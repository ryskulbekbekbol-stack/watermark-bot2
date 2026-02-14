import logging
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
from PIL import Image, ImageDraw, ImageFont
from io import BytesIO
import os

TOKEN = os.getenv("BOT_TOKEN")
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class WatermarkBot:
    def __init__(self):
        # Текст как на картинке
        self.watermark_lines = [
            "Tgk soon_na_volne",
            "Tgk soon_na_volne", 
            "Tgk soon_na_volne",
            "Tgksoon_na_volne",
            "Tgk soon_na_volne",
            "Tgk soon_ na_volne",
            "Tgk soon_na_volne",
            "Tgk soon_na_volne",
            "Tgk soon_na_ volne",
            "Tgk soon_na_volne",
            "Tgk soon_na_volne"
        ]
        
    def add_multiline_watermark(self, image_bytes):
        """Добавляет многострочный водяной знак как на картинке"""
        try:
            # Открываем изображение
            img = Image.open(BytesIO(image_bytes))
            if img.mode != 'RGB':
                img = img.convert('RGB')
            
            width, height = img.size
            
            # Создаем слой для водяного знака
            watermark = Image.new('RGBA', img.size, (255, 255, 255, 0))
            draw = ImageDraw.Draw(watermark)
            
            # Определяем размер шрифта в зависимости от размера изображения
            base_font_size = int(height * 0.03)  # 3% от высоты
            if base_font_size < 12:
                base_font_size = 12
            if base_font_size > 24:
                base_font_size = 24
            
            # Пробуем загрузить моноширинный шрифт (как на картинке)
            font = None
            font_paths = [
                '/system/fonts/DroidSansMono.ttf',
                '/system/fonts/RobotoMono-Regular.ttf',
                '/system/fonts/CutiveMono.ttf',
                '/system/fonts/DejaVuSansMono.ttf'
            ]
            
            for path in font_paths:
                try:
                    if os.path.exists(path):
                        font = ImageFont.truetype(path, base_font_size)
                        break
                except:
                    continue
            
            if font is None:
                # Если не нашли моноширинный, используем обычный
                try:
                    font = ImageFont.truetype('/system/fonts/Roboto-Regular.ttf', base_font_size)
                except:
                    font = ImageFont.load_default()
            
            # Рассчитываем межстрочный интервал
            try:
                # Получаем высоту одной строки
                bbox = draw.textbbox((0, 0), "Tg", font=font)
                line_height = bbox[3] - bbox[1]
            except:
                line_height = base_font_size + 4
            
            spacing = int(line_height * 1.2)  # Межстрочный интервал
            
            # Прозрачность (чем меньше - тем прозрачнее)
            opacity = 80  # Довольно прозрачный
            
            # Цвет текста - белый полупрозрачный
            text_color = (255, 255, 255, opacity)
            
            # Рисуем все строки
            for i, line in enumerate(self.watermark_lines):
                # Позиция - правая часть изображения, вертикально по центру
                x = width * 0.7  # 70% от ширины (правая часть)
                y = (height - len(self.watermark_lines) * spacing) // 2 + i * spacing
                
                # Рисуем текст
                draw.text((x, y), line, font=font, fill=text_color)
            
            # Также добавляем текст слева (для заполнения пространства)
            for i, line in enumerate(self.watermark_lines):
                x = width * 0.1  # 10% от ширины (левая часть)
                y = (height - len(self.watermark_lines) * spacing) // 2 + i * spacing
                draw.text((x, y), line, font=font, fill=text_color)
            
            # Накладываем водяной знак на изображение
            img_rgba = img.convert('RGBA')
            watermarked = Image.alpha_composite(img_rgba, watermark)
            watermarked = watermarked.convert('RGB')
            
            # Сохраняем результат
            output = BytesIO()
            watermarked.save(output, format='JPEG', quality=90)
            output.seek(0)
            
            return output.getvalue()
            
        except Exception as e:
            logger.error(f"Ошибка при создании водяного знака: {e}")
            return None
    
    def add_diagonal_multiline_watermark(self, image_bytes):
        """Вариант с диагональным расположением текста"""
        try:
            img = Image.open(BytesIO(image_bytes))
            if img.mode != 'RGB':
                img = img.convert('RGB')
            
            width, height = img.size
            
            # Создаем водяной знак на большем холсте для поворота
            watermark = Image.new('RGBA', (width * 2, height * 2), (255, 255, 255, 0))
            draw = ImageDraw.Draw(watermark)
            
            # Размер шрифта
            font_size = int(min(width, height) * 0.02)
            if font_size < 10:
                font_size = 10
            
            # Загружаем шрифт
            font = None
            try:
                font_paths = ['/system/fonts/DroidSansMono.ttf',
                             '/system/fonts/RobotoMono-Regular.ttf']
                for path in font_paths:
                    if os.path.exists(path):
                        font = ImageFont.truetype(path, font_size)
                        break
            except:
                font = ImageFont.load_default()
            
            if font is None:
                font = ImageFont.load_default()
            
            # Межстрочный интервал
            spacing = int(font_size * 1.5)
            
            # Прозрачность
            opacity = 60
            
            # Рисуем текст в центре увеличенного холста
            start_x = watermark.width // 4
            start_y = watermark.height // 4
            
            for i, line in enumerate(self.watermark_lines):
                x = start_x
                y = start_y + i * spacing
                draw.text((x, y), line, font=font, fill=(255, 255, 255, opacity))
            
            # Поворачиваем на 30 градусов
            watermark = watermark.rotate(30, expand=0, resample=Image.BICUBIC)
            
            # Обрезаем до исходного размера
            left = (watermark.width - width) // 2
            top = (watermark.height - height) // 2
            watermark = watermark.crop((left, top, left + width, top + height))
            
            # Накладываем
            img_rgba = img.convert('RGBA')
            result = Image.alpha_composite(img_rgba, watermark)
            result = result.convert('RGB')
            
            output = BytesIO()
            result.save(output, format='JPEG', quality=90)
            output.seek(0)
            
            return output.getvalue()
            
        except Exception as e:
            logger.error(f"Ошибка: {e}")
            return None
    
    def add_repeated_watermark(self, image_bytes):
        """Повторяющийся водяной знак по всей площади"""
        try:
            img = Image.open(BytesIO(image_bytes))
            if img.mode != 'RGB':
                img = img.convert('RGB')
            
            width, height = img.size
            
            watermark = Image.new('RGBA', img.size, (255, 255, 255, 0))
            draw = ImageDraw.Draw(watermark)
            
            # Маленький шрифт для повторения
            font_size = int(min(width, height) * 0.02)
            if font_size < 8:
                font_size = 8
            
            font = None
            try:
                font_paths = ['/system/fonts/DroidSansMono.ttf',
                             '/system/fonts/RobotoMono-Regular.ttf']
                for path in font_paths:
                    if os.path.exists(path):
                        font = ImageFont.truetype(path, font_size)
                        break
            except:
                font = ImageFont.load_default()
            
            if font is None:
                font = ImageFont.load_default()
            
            # Межстрочный интервал
            spacing = int(font_size * 1.8)
            
            # Прозрачность
            opacity = 40  # Более прозрачный
            
            # Рисуем сетку водяных знаков
            for y in range(0, height + spacing, spacing):
                for x in range(0, width, 300):  # Горизонтальный шаг
                    # Берем строки по очереди
                    line_idx = (y // spacing) % len(self.watermark_lines)
                    line = self.watermark_lines[line_idx]
                    
                    # Смещение для шахматного порядка
                    offset = (y // spacing) % 2 * 150
                    
                    draw.text((x + offset, y), line, font=font, 
                             fill=(255, 255, 255, opacity))
            
            img_rgba = img.convert('RGBA')
            result = Image.alpha_composite(img_rgba, watermark)
            result = result.convert('RGB')
            
            output = BytesIO()
            result.save(output, format='JPEG', quality=90)
            output.seek(0)
            
            return output.getvalue()
            
        except Exception as e:
            logger.error(f"Ошибка: {e}")
            return None

# Создаем экземпляр бота
bot = WatermarkBot()

# Обработчики Telegram
async def start_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Команда /start"""
    await update.message.reply_text(
        "🤖 Бот для добавления водяного знака\n\n"
        "Текст водяного знака:\n"
        "Tgk soon_na_volne\n"
        "(11 строк как на картинке)\n\n"
        "Отправьте фото - и я добавлю водяной знак!\n\n"
        "Режимы:\n"
        "/standard - стандартный (сбоку)\n"
        "/diagonal - диагональный\n"
        "/repeated - повторяющийся\n"
        "/custom - изменить текст"
    )

async def set_standard(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Установить стандартный режим"""
    context.user_data['mode'] = 'standard'
    await update.message.reply_text("✅ Режим: Стандартный (текст сбоку)")

async def set_diagonal(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Установить диагональный режим"""
    context.user_data['mode'] = 'diagonal'
    await update.message.reply_text("✅ Режим: Диагональный")

async def set_repeated(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Установить режим повторения"""
    context.user_data['mode'] = 'repeated'
    await update.message.reply_text("✅ Режим: Повторяющийся")

async def set_custom_text(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Изменить текст водяного знака"""
    if context.args:
        custom_text = " ".join(context.args)
        # Разбиваем на строки
        lines = custom_text.split('\\n')  # Пользователь вводит \n для новой строки
        if len(lines) == 1:
            # Если одна строка, повторяем ее 11 раз
            lines = [custom_text] * 11
        bot.watermark_lines = lines[:11]  # Берем первые 11 строк
        await update.message.reply_text(f"✅ Текст изменен на:\n" + "\n".join(bot.watermark_lines))
    else:
        await update.message.reply_text(
            "Введите текст водяного знака.\n"
            "Пример: /custom Первая строка\\nВторая строка\\nТретья строка\n"
            "(используйте \\n для новой строки)"
        )

async def handle_photo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработка фотографий"""
    try:
        await update.message.reply_text("🔄 Добавляю водяной знак...")
        
        # Получаем фото
        photo = update.message.photo[-1]
        photo_file = await photo.get_file()
        photo_bytes = await photo_file.download_as_bytearray()
        
        # Определяем режим
        mode = context.user_data.get('mode', 'standard')
        
        # Обрабатываем в зависимости от режима
        if mode == 'diagonal':
            result = bot.add_diagonal_multiline_watermark(bytes(photo_bytes))
        elif mode == 'repeated':
            result = bot.add_repeated_watermark(bytes(photo_bytes))
        else:  # standard
            result = bot.add_multiline_watermark(bytes(photo_bytes))
        
        if result:
            await update.message.reply_photo(
                photo=BytesIO(result),
                caption="✅ Готово! Водяной знак добавлен."
            )
        else:
            await update.message.reply_text("❌ Не удалось добавить водяной знак")
            
    except Exception as e:
        logger.error(f"Ошибка обработки фото: {e}")
        await update.message.reply_text(f"❌ Ошибка: {str(e)}")

async def show_text(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Показать текущий текст водяного знака"""
    text = "\n".join(bot.watermark_lines)
    await update.message.reply_text(f"Текущий текст водяного знака:\n\n{text}")

async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Команда /help"""
    await update.message.reply_text(
        "ℹ️ Помощь:\n\n"
        "1. Отправьте фото\n"
        "2. Бот добавит водяной знак\n\n"
        "Текст по умолчанию:\n"
        "Tgk soon_na_volne (11 строк)\n\n"
        "Команды:\n"
        "/start - информация\n"
        "/standard - текст сбоку\n"
        "/diagonal - диагональный\n"
        "/repeated - повторяющийся\n"
        "/custom - изменить текст\n"
        "/show - показать текущий текст\n"
        "/help - эта справка"
    )

def main():
    """Запуск бота"""
    print("=" * 50)
    print("🤖 TELEGRAM WATERMARK BOT")
    print("=" * 50)
    print("Текст водяного знака:")
    for line in bot.watermark_lines:
        print(f"  {line}")
    print("=" * 50)
    
    if TOKEN == "ВАШ_ТОКЕН_БОТА_ЗДЕСЬ":
        print("\n❌ Замените TOKEN на настоящий токен!")
        print("\nКак получить токен:")
        print("1. В Telegram найдите @BotFather")
        print("2. Отправьте /newbot")
        print("3. Создайте бота и скопируйте токен")
        return
    
    try:
        # Создаем Application
        application = Application.builder().token(TOKEN).build()
        
        # Добавляем обработчики команд
        application.add_handler(CommandHandler("start", start_command))
        application.add_handler(CommandHandler("help", help_command))
        application.add_handler(CommandHandler("standard", set_standard))
        application.add_handler(CommandHandler("diagonal", set_diagonal))
        application.add_handler(CommandHandler("repeated", set_repeated))
        application.add_handler(CommandHandler("custom", set_custom_text))
        application.add_handler(CommandHandler("show", show_text))
        
        # Добавляем обработчик фото
        application.add_handler(MessageHandler(filters.PHOTO, handle_photo))
        
        # Обработчик текстовых сообщений
        application.add_handler(MessageHandler(
            filters.TEXT & ~filters.COMMAND,
            lambda u, c: u.message.reply_text("📸 Отправьте фото для добавления водяного знака!")
        ))
        
        print("\n✅ Бот запущен!")
        print("📱 Отправьте фото вашему боту")
        print("=" * 50)
        
        # Запускаем бота
        application.run_polling()
        
    except Exception as e:
        print(f"\n❌ Ошибка запуска: {e}")

if __name__ == '__main__':
    main()
