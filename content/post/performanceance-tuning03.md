---
title: "jstack 死循环和死锁定位"
date: "2018-09-03 22:26:21"
tags: ["性能调优"]
categories: ["架构"]
---
jstack 是用来查看线程的命令
```bash
jstack [option] <pid>
```

先来看看线程的状态([官方文档](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr034.html#sthref31))
- New
- Runnable
- Blocked
- Waiting
- Timed_Waiting
- Terminated

下面是 java 线程状态转化
![thread](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-07-08-030046.jpg)


代码模拟
```java
@RestController
public class CpuController {
	
	/**
	 * 死循环
	 * */
	@RequestMapping("/loop")
	public List<Long> loop(){
		String data = "{\"data\":[{\"partnerid\":]";
		return getPartneridsFromJson(data);
	}
	
	private Object lock1 = new Object();
	private Object lock2 = new Object();
	
	/**
	 * 死锁
	 * */
	@RequestMapping("/deadlock")
	public String deadlock(){
		new Thread(()->{
			synchronized(lock1) {
				try {Thread.sleep(1000);}catch(Exception e) {}
				synchronized(lock2) {
					System.out.println("Thread1 over");
				}
			}
		}) .start();
		new Thread(()->{
			synchronized(lock2) {
				try {Thread.sleep(1000);}catch(Exception e) {}
				synchronized(lock1) {
					System.out.println("Thread2 over");
				}
			}
		}) .start();
		return "deadlock";
	}
	public static List<Long> getPartneridsFromJson(String data){  
	    //{\"data\":[{\"partnerid\":982,\"count\":\"10000\",\"cityid\":\"11\"},{\"partnerid\":983,\"count\":\"10000\",\"cityid\":\"11\"},{\"partnerid\":984,\"count\":\"10000\",\"cityid\":\"11\"}]}  
	    //上面是正常的数据  
	    List<Long> list = new ArrayList<Long>(2);  
	    if(data == null || data.length() <= 0){  
	        return list;  
	    }      
	    int datapos = data.indexOf("data");  
	    if(datapos < 0){  
	        return list;  
	    }  
	    int leftBracket = data.indexOf("[",datapos);  
	    int rightBracket= data.indexOf("]",datapos);  
	    if(leftBracket < 0 || rightBracket < 0){  
	        return list;  
	    }  
	    String partners = data.substring(leftBracket+1,rightBracket);  
	    if(partners == null || partners.length() <= 0){  
	        return list;  
	    }  
	    while(partners!=null && partners.length() > 0){  
	        int idpos = partners.indexOf("partnerid");  
	        if(idpos < 0){  
	            break;  
	        }  
	        int colonpos = partners.indexOf(":",idpos);  
	        int commapos = partners.indexOf(",",idpos);  
	        if(colonpos < 0 || commapos < 0){  
	            //partners = partners.substring(idpos+"partnerid".length());//1  
	            continue;
	        }  
	        String pid = partners.substring(colonpos+1,commapos);  
	        if(pid == null || pid.length() <= 0){  
	            //partners = partners.substring(idpos+"partnerid".length());//2  
	            continue;
	        }  
	        try{  
	            list.add(Long.parseLong(pid));  
	        }catch(Exception e){  
	            //do nothing  
	        }  
	        partners = partners.substring(commapos);  
	    }  
	    return list;  
	}  
	
}

```
### 死循环定位
```bash
top 
jstack <pid> > result.txt #将结果追加到 txt 文件


```
负载过大会导致无法访问，使用 top 命名找到占用 cpu 最多的那个进程，
```
top -p <pid> -H #打印所有的线程
printf "%x" <pid> 将十进制的 pid 转换成 十六进制
找到线程对应的 pid 
```

### 死锁定位
```bash
jps -l #或者 ps -ef | grep java
jstack <pid> > result.txt
```
如果有死锁则会在文件末尾出现提示。