---
title: "gitlab 添加代码规范检测"
date: "2019-06-19 11:00:46"
tags: ["devops"]
categories: ["架构"]
description: "在推代码规范的路上一定要独裁"
---

# 环境

gitlab 10.5.X + gitlab + ali p3c.pmd

好的代码能让人赏心悦目，方便 CodeReview 进行，为此，必须强制性的将代码规范起来。

# 如何做

## git custom hooks

### client hooks

客户端钩子是存在本地的，按项目去配，这样能达到目的，但是起不到强制约束的效果，而且一个一个配很麻烦

### server hooks

服务器端的钩子，每个项目中的 git hooks 全都是软连接到 gitlab  上的一个位置，默认是在

```bash
/opt/gitlab/embedded/service/gitlab-shell/hooks
```

这个 hooks 文件夹内包括三个文件

```bash
.
├── post-receive
├── pre-receive
└── update

```

- pre-receive 接收客户端推送的代码，这个脚本的功能是最后只有 exit 0 时才代表接收，所有非 0 的推送都会被拒绝
- update 可以多分支的去检测，即一段代码推送到多个分支，每个分支更新代码前都会执行一次这个脚本
- post-receive 这个脚本是最后执行的，主要用来记录信息及通知用，不能中断推送流程

按实际需求，使用 pre-receive 脚本就能满足要求，修改脚本为如下内容：

```bash
#!/bin/sh
export LANG="en_US.UTF-8"
REJECT=0

while read oldrev newrev refname; do
    if [ "$oldrev" = "0000000000000000000000000000000000000000" ];then
        oldrev="${newrev}^"
    fi

    files=`git diff --name-only ${oldrev} ${newrev}  | grep -e "\.java$"`

    if [ -n "$files" ]; then
        TEMPDIR="tmp"
        for file in ${files}; do
            mkdir -p "${TEMPDIR}/`dirname ${file}`" >/dev/null
            git show $newrev:$file > ${TEMPDIR}/${file}
        done;

        files_to_check=`find $TEMPDIR -name '*.java'`
        
        
         /home/jdk1.8.0_212/bin/java -cp /home/p3c-pmd-1.3.6.jar net.sourceforge.pmd.PMD -d $TEMPDIR -R rulesets/java/ali-comment.xml,rulesets/java/ali-concurrent.xml,rulesets/java/ali-constant.xml,rulesets/java/ali-exception.xml,rulesets/java/ali-flowcontrol.xml,rulesets/java/ali-naming.xml,rulesets/java/ali-oop.xml,rulesets/java/ali-orm.xml,rulesets/java/ali-other.xml,rulesets/java/ali-set.xml -f text
       
         REJECT=$?
         
         
          if [[ $REJECT == 0 ]] ;then
            echo -e "\033[32m恭喜你代码通过质量检测！\033[0m"
             else echo -e "\033[31m\033[01m 请及时修改代码并再次尝试\033[0m" 
          fi
     
        rm -rf $TEMPDIR
    fi
done

exit $REJECT
```

上面代码是根据简书上的一个 blog 做了微调，如果你要用的话只需要修改 java 路径和 p3c.jar 的路径，这个脚本接受的是标准输入流，具体可以查看下方链接。

参考链接：

[如何生成整合了阿里巴巴JAVA编码规范的PMD包配合GitLab提升团队代码质量](https://www.jianshu.com/p/b87ca8615c9c)

[Gitlab 服务器端 custom hook 配置](<https://www.jianshu.com/p/5531a21afa68>)

[自定义 Git - Git 钩子]([https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90](https://git-scm.com/book/zh/v2/自定义-Git-Git-钩子))

[githooks(5) Manual Page](<https://gitirc.eu/githooks.html>)

[使用git钩子对提交代码进行检查(pre-commit)](<https://www.jianshu.com/p/de90ffbd53e9>)

