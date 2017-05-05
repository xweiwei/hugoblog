+++
css = []
date = "2017-05-05T22:43:28+08:00"
description = ""
draft = false
highlight = true
scripts = []
tags = ["linux", "Android", "CVE"]
title = "CVE-2016-10229分析"
index = true
+++




# 漏洞描述

Linux kernel是美国Linux基金会发布的操作系统Linux所使用的内核。Linux kernel 4.5之前的版本中的udp.c文件存在安全漏洞，Linux内核中的udp.c允许远程攻击者通过UDP流量执行任意代码，这些流量会在执行具有MSG_PEEK标志的recv系统调用时触发不安全的第二次校验和计算，远程攻击者可精心构造数据执行任意代码，进一步导致本地提权，属于高危漏洞。但由于现实情况中，基于UDP协议的服务时MSG_PEEK标志在实际使用的情况较少，受该远程命令执行漏洞危害影响群体范围有限。

该漏洞是来自谷歌的Eric Dumazet发现的，他说漏洞源于2015年年末的一个Linux内核补丁。


# 代码分析


`*__skb_recv_datagram`里判断peek，会使得skb->users加1.

`https://android.googlesource.com/kernel/msm/+/android-7.1.2_r0.7/net/core/datagram.c`

```
194			*peeked = skb->peeked;
195			if (flags & MSG_PEEK) {
196				if (_off >= skb->len && (skb->len || _off ||
197							 skb->peeked)) {
198					_off -= skb->len;
199					continue;
200				}
201				skb->peeked = 1;
202				atomic_inc(&skb->users);
```

skb_shared(skb)为假，skb被多个对象引用，不会将csum结果存到skb上，所以后面的代码会再次计算。
`https://android.googlesource.com/kernel/msm/+/android-7.1.2_r0.7/net/core/datagram.c`
```
__sum16 __skb_checksum_complete(struct sk_buff *skb)
{
	__wsum csum;
	__sum16 sum;
	csum = skb_checksum(skb, 0, skb->len, 0);
	/* skb->csum holds pseudo checksum */
	sum = csum_fold(csum_add(skb->csum, csum));
	if (likely(!sum)) {
		if (unlikely(skb->ip_summed == CHECKSUM_COMPLETE) &&
		    !skb->csum_complete_sw)
			netdev_rx_csum_fault(skb->dev);
	}
	if (!skb_shared(skb)) {
		/* Save full packet checksum */
		skb->csum = csum;
		skb->ip_summed = CHECKSUM_COMPLETE;
		skb->csum_complete_sw = 1;
		skb->csum_valid = !sum;
	}
	return sum;
}

```


```
1401 /**
1402  *      skb_shared - is the buffer shared
1403  *      @skb: buffer to check
1404  *
1405  *      Returns true if more than one person has a reference to this
1406  *      buffer.
1407  */
1408 static inline int skb_shared(const struct sk_buff *skb)
1409 {
1410         return atomic_read(&skb->users) != 1;
1411 }
```

这里`udp_lib_checksum_complete`会去计算一次校验和，然后`skb_csum_unnecessary`会去判断是否计算过，但是由于MSG_PEEK的原因，会跳到`skb_copy_and_csum_datagram_iovec`。

```

1279	if (copied < ulen || UDP_SKB_CB(skb)->partial_cov) {
1280		if (udp_lib_checksum_complete(skb))
1281			goto csum_copy_err;
1282	}
1283
1284	if (skb_csum_unnecessary(skb))
1285		err = skb_copy_datagram_iovec(skb, sizeof(struct udphdr),
1286					      msg->msg_iov, copied);
1287	else {
1288		err = skb_copy_and_csum_datagram_iovec(skb,
1289						       sizeof(struct udphdr),
1290						       msg->msg_iov);
1291
1292		if (err == -EINVAL)
1293			goto csum_copy_err;
1294	}

2875static inline int skb_csum_unnecessary(const struct sk_buff *skb)
2876{
2877	return ((skb->ip_summed & CHECKSUM_UNNECESSARY) || skb->csum_valid);
2878}

```

`skb_copy_and_csum_datagram_iovec`没有考虑用户层buf的长度，在复制时会出现越界。所以有问题不是因为计算校验和有问题，而是因为跳到了一个不安全的复制函数里（而跳到这里的结果是第一次计算的校验和结果没有存，导致需要再计算）。这里由于`iov->iov_len < chunk`所以会到`skb_copy_datagram_iovec`。

```
/**
 *	skb_copy_and_csum_datagram_iovec - Copy and checksum skb to user iovec.
 *	@skb: skbuff
 *	@hlen: hardware length
 *	@iov: io vector
 *
 *	Caller _must_ check that skb will fit to this iovec.
 *
 *	Returns: 0       - success.
 *		 -EINVAL - checksum failure.
 *		 -EFAULT - fault during copy. Beware, in this case iovec
 *			   can be modified!
 */
int skb_copy_and_csum_datagram_iovec(struct sk_buff *skb,
				     int hlen, struct iovec *iov)
{
	__wsum csum;
	int chunk = skb->len - hlen;
	if (!chunk)
		return 0;
	/* Skip filled elements.
	 * Pretty silly, look at memcpy_toiovec, though 8)
	 */
	while (!iov->iov_len)
		iov++;
	if (iov->iov_len < chunk) {
		if (__skb_checksum_complete(skb))
			goto csum_error;
		if (skb_copy_datagram_iovec(skb, hlen, iov, chunk))
			goto fault;
	} else {
		csum = csum_partial(skb->data, hlen, skb->csum);
		if (skb_copy_and_csum_datagram(skb, hlen, iov->iov_base,
					       chunk, &csum))
			goto fault;
		if (csum_fold(csum))
			goto csum_error;
		if (unlikely(skb->ip_summed == CHECKSUM_COMPLETE))
			netdev_rx_csum_fault(skb->dev);
		iov->iov_len -= chunk;
		iov->iov_base += chunk;
	}
	return 0;
csum_error:
	return -EINVAL;
fault:
	return -EFAULT;
}
```


```

311/**
312 *	skb_copy_datagram_iovec - Copy a datagram to an iovec.
313 *	@skb: buffer to copy
314 *	@offset: offset in the buffer to start copying from
315 *	@to: io vector to copy to
316 *	@len: amount of data to copy from buffer to iovec
317 *
318 *	Note: the iovec is modified during the copy.
319 */
320int skb_copy_datagram_iovec(const struct sk_buff *skb, int offset,
321			    struct iovec *to, int len)
322{
323	int start = skb_headlen(skb);
324	int i, copy = start - offset;
325	struct sk_buff *frag_iter;
326
327	trace_skb_copy_datagram_iovec(skb, len);
328
329	/* Copy header. */ //复制udp头
330	if (copy > 0) {
331		if (copy > len)
332			copy = len;
333		if (memcpy_toiovec(to, skb->data + offset, copy))
334			goto fault;
335		if ((len -= copy) == 0)
336			return 0;
337		offset += copy;
338	}
339
340	/* Copy paged appendix. Hmm... why does this look so complicated? */ 
341	for (i = 0; i < skb_shinfo(skb)->nr_frags; i++) {
342		int end;
343		const skb_frag_t *frag = &skb_shinfo(skb)->frags[i];
344
345		WARN_ON(start > offset + len);
346
347		end = start + skb_frag_size(frag);
348		if ((copy = end - offset) > 0) {
349			int err;
350			u8  *vaddr;
351			struct page *page = skb_frag_page(frag);
352
353			if (copy > len)
354				copy = len;
355			vaddr = kmap(page);
356			err = memcpy_toiovec(to, vaddr + frag->page_offset +
357					     offset - start, copy);
358			kunmap(page);
359			if (err)
360				goto fault;
361			if (!(len -= copy))
362				return 0;
363			offset += copy;
364		}
365		start = end;
366	}
367
368	skb_walk_frags(skb, frag_iter) { //复制分片
369		int end;
370
371		WARN_ON(start > offset + len);
372
373		end = start + frag_iter->len;
374		if ((copy = end - offset) > 0) {
375			if (copy > len)
376				copy = len;
377			if (skb_copy_datagram_iovec(frag_iter,
378						    offset - start,
379						    to, copy))
380				goto fault;
381			if ((len -= copy) == 0)
382				return 0;
383			offset += copy;
384		}
385		start = end;
386	}
387	if (!len)
388		return 0;
389
390fault:
391	return -EFAULT;
392}
393EXPORT_SYMBOL(skb_copy_datagram_iovec);

```


`memcpy_toiovec`会通过copy_to_user向用户态的iov写数据。memcpy_toiovec会去判断iov->iov_len的剩余长度，并且只将数据放在剩余的空间里，那又如何越界？ 但是这里不会去判断iov的个数，这里可以造成越界写用户数据。


```
30/*
31 *	Copy kernel to iovec. Returns -EFAULT on error.
32 *
33 *	Note: this modifies the original iovec.
34 */
35
36int memcpy_toiovec(struct iovec *iov, unsigned char *kdata, int len)
37{
38	while (len > 0) {
39		if (iov->iov_len) {
40			int copy = min_t(unsigned int, iov->iov_len, len);
41			if (copy_to_user(iov->iov_base, kdata, copy))
42				return -EFAULT;
43			kdata += copy;
44			len -= copy;
45			iov->iov_len -= copy;
46			iov->iov_base += copy;
47		}
48		iov++;
49	}
50
51	return 0;
52}
53EXPORT_SYMBOL(memcpy_toiovec);
```

在/include/linux/kernel.h
```
746#define min_t(type, x, y) ({			\
747	type __min1 = (x);			\
748	type __min2 = (y);			\
749	__min1 < __min2 ? __min1: __min2; })
750
```


# 引起

由另外一个patch引起
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=89c22d8c3b27
```

diff --git a/net/core/datagram.c b/net/core/datagram.c
index 4e9a3f6..4967262 100644
--- a/net/core/datagram.c
+++ b/net/core/datagram.c
@@ -657,7 +657,8 @@ __sum16 __skb_checksum_complete_head(struct sk_buff *skb, int len)
 		    !skb->csum_complete_sw)
 			netdev_rx_csum_fault(skb->dev);
 	}
-	skb->csum_valid = !sum;
+	if (!skb_shared(skb))
+		skb->csum_valid = !sum;
 	return sum;
 }
 EXPORT_SYMBOL(__skb_checksum_complete_head);
@@ -677,11 +678,13 @@ __sum16 __skb_checksum_complete(struct sk_buff *skb)
 			netdev_rx_csum_fault(skb->dev);
 	}
 
-	/* Save full packet checksum */
-	skb->csum = csum;
-	skb->ip_summed = CHECKSUM_COMPLETE;
-	skb->csum_complete_sw = 1;
-	skb->csum_valid = !sum;
+	if (!skb_shared(skb)) {
+		/* Save full packet checksum */
+		skb->csum = csum;
+		skb->ip_summed = CHECKSUM_COMPLETE;
+		skb->csum_complete_sw = 1;
+		skb->csum_valid = !sum;
+	}
 
 	return sum;
 }
```

# 利用方式
远程执行确实可以做到，可以在写用户态。通过root权限的应用，也可以提权，但是在内核态下不能做到代码任意执行。

# 参考

[CVE-2016-10229 Linux remote code execution flaw potentially exposes systems at risk of hack](http://securityaffairs.co/wordpress/57998/hacking/cve-2016-10229-linux.html)

[UDP 收发](http://blog.csdn.net/qy532846454/article/details/7010852)

[UDP 校验和](http://blog.csdn.net/qy532846454/article/details/6993695)

[struct sk_buff结构体详解](http://weiguozhihui.blog.51cto.com/3060615/1586777)

[msghdr 结构体](http://blog.csdn.net/ccccdddxxx/article/details/6368337)

提到的另一个修改，增加了一个用户层buf长度的参数。
https://patchwork.ozlabs.org/patch/530642/


