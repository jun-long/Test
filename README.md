import os
import shutil
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders

# 文件夹路径
source_dir = "/home/a/b/c"
dest_dir = "/home/a/b"
zip_name = "images_archive"  # 压缩包名（不带扩展名）
zip_path = os.path.join(dest_dir, zip_name)

# 邮件配置
smtp_server = "smtp.example.com"
smtp_port = 587
email_user = "your_email@example.com"
email_password = "your_password"
recipient = "recipient_email@example.com"

def compress_images():
    """压缩 /home/a/b/c 目录中的所有图片"""
    # 创建压缩包，保存为 /home/a/b/images_archive.zip，结构为 c/所有图片
    shutil.make_archive(zip_path, 'zip', root_dir=source_dir, base_dir=".")
    print(f"压缩完成，文件保存在: {zip_path}.zip")

def send_email():
    """发送邮件并附加压缩包"""
    # 创建邮件对象
    msg = MIMEMultipart()
    msg['From'] = email_user
    msg['To'] = recipient
    msg['Subject'] = "图片压缩包"

    # 邮件正文
    body = "您好，请查收附件中的图片压缩包。"
    msg.attach(MIMEText(body, 'plain'))

    # 添加压缩包作为附件
    with open(f"{zip_path}.zip", "rb") as attachment:
        part = MIMEBase('application', 'octet-stream')
        part.set_payload(attachment.read())
        encoders.encode_base64(part)
        part.add_header(
            "Content-Disposition",
            f"attachment; filename={zip_name}.zip",
        )
        msg.attach(part)

    # 连接 SMTP 服务器并发送邮件
    try:
        server = smtplib.SMTP(smtp_server, smtp_port)
        server.starttls()  # 启用 TLS
        server.login(email_user, email_password)
        server.sendmail(email_user, recipient, msg.as_string())
        print("邮件发送成功！")
    except Exception as e:
        print(f"邮件发送失败: {e}")
    finally:
        server.quit()

if __name__ == "__main__":
    # 压缩图片
    compress_images()
    
    # 发送邮件
    send_email()
