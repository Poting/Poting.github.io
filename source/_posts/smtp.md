---
title: ASP.NET Core 2.0 Web API寄信功能
date: 2017-10-31
---
>值得紀念的第一篇文章~~~~  
因為最近在玩Web API 的寄信功能，就隨手紀錄吧


# SMTP
在 ASP .NET Core 1.x 時並沒有包含SMTP，所以就需要安裝一些套件來達成寄信的功能，像是
**MailKit**、**Mailgun**等等之類的，到了2.0時包進了 SMTP，這真的hen棒!! (可以直接copy code過來修改)。  

好的，所以什麼是SMTP?
>簡單郵件傳輸協定 (Simple Mail Transfer Protocol) 是事實上的在Internet傳輸email的標準

根據WIKI 的定義非常的淺顯易懂，就是一個處理郵件的協定，電腦上安裝了 SMTP 之後就可以寄信拉，通常都是透過**port 25**來建立連線，正常來說應該不會有程式占用 25 port 不過保險起見，還是確認一下比較好。  

SMTP是一個 "推送" 的協定，如果要接收信件，就需要 **POP3**、或**IMAP**來實作，
但是這不是這次的重點，所以改天再來說吧。  

下面列出各家常用的SMTP address  

![image](https://i.imgur.com/GLCbohj.png)

## 補充
如果用 SMTP 寄出的信件沒有寄件備份，找郵件管理員大大作設定 :) 不過用資料庫存起來，後續的資料處理 ( 應該 ) 會比較方便吧 ? (應該...)

# 範例
來點sample code吧，首先在 Repository 新增一個 Method 來寄信

``` C
public void Send_mail()
        {
            string mail_server = "Your Server IP";
            string mail_subject = "This is Title";
            string mail_body = "";
            mail_body += "<h3>Dear 測試信件</h3>";
            mail_body += "<p>Here is a LOOOONG Mail Body：<p>";
            mail_body += "<br>";
            mail_body += "<p>";
            mail_body += "</p>";

            string mail_sender = "Send@email.com";
            string address = "Receive@email.com";
            <!-- (寄信者, 收信者) -->
            MailMessage message = new MailMessage(mail_sender, address);
            message.IsBodyHtml = true;
            <!-- E-mail編碼 -->
            message.BodyEncoding = System.Text.Encoding.UTF8;
            <!-- E-mail主旨 -->
            message.Subject = mail_subject;
            <!-- 優先權 -->
            message.Priority = MailPriority.Normal; 
            <!-- E-mail內容 -->
            message.Body = mail_body;

            SmtpClient smtpClient = new SmtpClient(mail_server, 25);//設定E-mail Server和port
            smtpClient.Credentials = new NetworkCredential("account", "password");
            smtpClient.Send(message);
        }
```

然後在 service 裡面就可以直接呼叫剛剛在 repository 新增的 method，完成

``` C
public void Email() {
    RegistRepo.Send_mail();
}
```
controller 就不多贅述，google吧。


>參考來源
 * http://learn-web-hosting-domain-name.mygreatname.com/how-mail-server-works/how-smtp-pop3-mail-servers-works.html
 * https://zh.wikipedia.org/wiki/%E7%AE%80%E5%8D%95%E9%82%AE%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE 
 * https://www.lifewire.com/what-are-the-gmail-smtp-settings-1170854
 * https://www.lifewire.com/what-are-yahoo-smtp-settings-for-email-1170875
 * https://www.lifewire.com/what-are-windows-live-hotmail-smtp-settings-1170861


