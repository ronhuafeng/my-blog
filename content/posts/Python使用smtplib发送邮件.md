---
title: Python使用smtplib发送邮件
id: 78
categories:
  - 技术记录
date: 2012-11-13 10:50:48
tags:
---

from smtplib import SMTP

#这些库是邮件格式使用的
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart

smtp = SMTP()

smtp.connect('smtp.163.com')

#这句话是必不可少的
smtp.helo()

#选择认证方式
smtp.esmtp_features['auth'] = 'LOGIN'
smtp.login('####@163.com', 'password')

message = MIMEMultipart()
message.attach(MIMEText('content'))
message["Subject"] = 'subject'

smtp.sendmail('####@163.com', ['mail_to@163.com'], message.as_string())

差不多邮件这样就可以发送出去了