1.consumer提交消费信息的时候是批量提交的，比如这个时候consumer机器重启了，正常情况是consumer会告诉broker说我上次消费到offset=123的地方了，但是如果是批量提交offset的时候重启了，那么这次提交就失败了，并且也不会被记录。等下次消费的时候，还是从批量提交之前的offset开始消费的。

2.设置enable.auto.commit=false，避免consumer自动提交offset，因为自动提交失败的话，会导致offset更新也失败，我们要保证的是及时消费失败了也能正确更新offset