# Email-Service-Asp.NetCore-Web-API
*Email service configuration in web api*

---

### *1. Set-up SmtpPassword in gmail.com*

#### *Go to gmail.com and click profile then go to Account*

![Step 1](https://github.com/Jesc06/Email-Service-Asp.NetCore-Web-API/blob/main/services/GoToAccount.png)

#### *Find Security then go to security*

![Step 1](https://github.com/Jesc06/Email-Service-Asp.NetCore-Web-API/blob/main/services/GoToSecurity.png)

#### *Find App Password then set-up your personal app password*

![Step 1](https://github.com/Jesc06/Email-Service-Asp.NetCore-Web-API/blob/main/services/GoToAppPassword.png)

---
### *2. Create application.json EmailSettings*

   ```json
    "EmailSettings": {
      "SmtpServer": "smtp.gmail.com",
      "SmtpPort": 587,
      "SmtpUser": "your email",
      "SmtpPassword": "your smtp password"
    }
   ```

---

### *3. Create Application Class library then create sub folder for Interfaces, DTO, Services*

#### * Create Interface*

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace RecordManagementSystem.Application.Features.OTP.Interfaces
{
    public interface IEmailService
    {
        Task SendEmailAsync(string toEmail, string subject, int message);
    }
}

```

#### *Create DTO*

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace RecordManagementSystem.Application.Features.OTP.DTO
{
    public class EmailSettingsDTO
    {
        public string SmtpServer { get; set; }
        public int SmtpPort { get; set; }
        public string SmtpUser { get; set; }
        public string SmtpPassword { get; set; }
    }
}

```


#### *Go to Program.cs then configure this*

```csharp

//Email Service
builder.Services.Configure<EmailSettingsDTO>(
    builder.Configuration.GetSection("EmailSettings")
);
```


#### *Create Services*

```csharp

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using RecordManagementSystem.Application.Features.OTP.DTO;
using RecordManagementSystem.Application.Features.OTP.Interfaces;
using Microsoft.Extensions.Options;
using MimeKit;
using MailKit.Net.Smtp;

namespace RecordManagementSystem.Application.Features.OTP.Services
{
    public class EmailService : IEmailService
    {
        private readonly EmailSettingsDTO _settings;
        public EmailService(IOptions<EmailSettingsDTO> settings)
        {
            _settings = settings.Value;
        }

        public async Task SendEmailAsync(string toEmail, string subject, int message)
        {
            var email = new MimeMessage();
            email.From.Add(new MailboxAddress("MinSUTrails", _settings.SmtpUser));
            email.To.Add(new MailboxAddress("", toEmail));
            email.Subject = subject;
            email.Body = new TextPart(MimeKit.Text.TextFormat.Html) { Text = message.ToString()};

            using var smtp = new SmtpClient();
            await smtp.ConnectAsync(_settings.SmtpServer, _settings.SmtpPort, MailKit.Security.SecureSocketOptions.StartTls);
            await smtp.AuthenticateAsync(_settings.SmtpUser, _settings.SmtpPassword);
            await smtp.SendAsync(email);
            await smtp.DisconnectAsync(true);
        }

    }
}


```


#### *Then after created services layer go to Program.cs again then configure this*

```csharp
builder.Services.AddScoped<IEmailService, EmailService>();
```




