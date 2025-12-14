import smtplib
import ssl
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email import encoders
import os
from typing import List, Union

# --- КОНФИГУРАЦИЯ (Замените своими данными) ---
SMTP_SERVER = 'smtp.gmail.com'
SMTP_PORT = 587
EMAIL_FROM = 'ваш_адрес@gmail.com'     
EMAIL_PASSWORD = 'ваш_пароль_приложения' # Используйте пароль приложения!
# ---------------------------------------------

def send_complex_email(
    subject: str, 
    body_text: str, 
    body_html: str,
    to_emails: Union[str, List[str]], 
    attachment_paths: Union[str, List[str], None] = None
) -> bool:
    """
    Отправляет сложное электронное письмо с HTML-контентом и множеством вложений/получателей.
    """
    
    # Нормализация списка получателей
    if isinstance(to_emails, str):
        recipient_list = [to_emails]
    else:
        recipient_list = to_emails
        
    # Создание контейнера MIMEMultipart (Alternative) для текстовой и HTML-версий
    msg = MIMEMultipart('alternative')
    msg['From'] = EMAIL_FROM
    msg['To'] = ", ".join(recipient_list) # Отображение списка получателей в заголовке
    msg['Subject'] = subject

    # Добавляем текстовую версию (важно для старых клиентов и фильтров спама)
    msg.attach(MIMEText(body_text, 'plain', 'utf-8'))
    
    # Добавляем HTML-версию (для форматирования)
    msg.attach(MIMEText(body_html, 'html', 'utf-8'))

    # 2. Прикрепление файлов
    if attachment_paths:
        if isinstance(attachment_paths, str):
            attachment_paths = [attachment_paths]
            
        for path in attachment_paths:
            if os.path.exists(path):
                attach_file(msg, path)
            else:
                print(f"⚠️ Вложение не найдено и пропущено: {path}")

    # 3. Подключение к SMTP-серверу
    context = ssl.create_default_context() # Обеспечивает безопасное соединение
    
    try:
        with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
            
            server.starttls(context=context) # Начинаем TLS-шифрование
            server.login(EMAIL_FROM, EMAIL_PASSWORD)
            
            # Отправка письма всем получателям
            server.sendmail(EMAIL_FROM, recipient_list, msg.as_string())
            
            print(f"✅ Письмо успешно отправлено {len(recipient_list)} получателям.")
            return True

    except smtplib.SMTPAuthenticationError:
        print("❌ Ошибка аутентификации. Проверьте пароль приложения.")
    except Exception as e:
        print(f"❌ Произошла ошибка при отправке почты: {e}")
    
    return False


def attach_file(msg, filename):
    """Добавляет файл к объекту MIME-сообщения."""
    try:
        with open(filename, "rb") as attachment:
            part = MIMEBase("application", "octet-stream")
            part.set_payload(attachment.read())

        encoders.encode_base64(part)

        part.add_header(
            "Content-Disposition",
            f"attachment; filename= {os.path.basename(filename)}",
        )

        msg.attach(part)
        print(f"Файл '{os.path.basename(filename)}' прикреплен.")

    except Exception as e:
        print(f"Ошибка при прикреплении файла {filename}: {e}")


# --- Использование ---
if __name__ == "__main__":
    
    # Создаем фиктивные файлы для примера
    dummy_file_1 = "report_1.pdf"
    dummy_file_2 = "image_data.png"
    with open(dummy_file_1, "w") as f: f.write("Отчет PDF.")
    with open(dummy_file_2, "w") as f: f.write("Двоичные данные.")

    subject = "Ежемесячная рассылка: Обновление финансового рынка"
    
    body_text = """
    Здравствуйте!
    
    В приложении вы найдете PDF-отчет. 
    Наш сайт: http://example.com/finance
    
    С уважением, Команда.
    """
    
    body_html = """
    <html>
        <head>
            <style>
                body { font-family: Arial, sans-serif; }
                .header { color: #1e88e5; font-size: 20px; }
            </style>
        </head>
        <body>
            <div class="header">📈 Обновление рынка за Декабрь</div>
            <p>Здравствуйте,</p>
            <p>Наш <a href="http://example.com/finance">финансовый отчет</a> ждет вас во вложении. Мы подготовили стильный PDF и аналитику рынка.</p>
            <p>С уважением,<br>Команда.</p>
        </body>
    </html>
    """
    
    # Отправляем письмо нескольким получателям с двумя вложениями
    recipients = ["первый_получатель@example.com", "второй_получатель@example.com"]
    attachments = [dummy_file_1, dummy_file_2]

    # Замените 'EMAIL_TO' на 'recipients' в реальном коде
    # send_complex_email(
    #     subject=subject,
    #     body_text=body_text,
    #     body_html=body_html,
    #     to_emails=recipients,
    #     attachment_paths=attachments
    # )

    print("Закомментированный вызов функции. Заполните переменные для запуска.")

    # Удаление фиктивных файлов
    if os.path.exists(dummy_file_1): os.remove(dummy_file_1)
    if os.path.exists(dummy_file_2): os.remove(dummy_file_2)
