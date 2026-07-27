import asyncio
from aiogram import Bot, Dispatcher, types, F
from aiogram.filters import Command
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.utils.keyboard import InlineKeyboardBuilder

# ================= НАСТРОЙКИ =================
BOT_TOKEN = "8961954979:AAHiAt6GV3uDFkd_YOLJXDJRjvDPceZNcBA"
ADMIN_ID = 8771237003  # ТВОЙ ЦИФРОВОЙ ID (без кавычек)
# =============================================

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher()

# Состояния для сбора данных (машина состояний)
class UserState:
    role = None
    name = None
    experience = None
    reason = None
    profile_link = None

user_data = {} # Хранилище временных данных пользователей: {user_id: {role, name, ...}}

# Клавиатура с ролями
def get_roles_keyboard():
    builder = InlineKeyboardBuilder()
    roles = ["Стажер", "Модер", "Куратор", "Админ", "Хелпер", "Медия"]
    for role in roles:
        builder.button(text=role, callback_data=f"role_{role}")
    return builder.as_markup()

# Клавиатура для админа (Одобрить/Отклонить)
def get_admin_keyboard(user_id, role):
    builder = InlineKeyboardBuilder()
    # Передаем user_id и role через callback_data
    builder.button(text="✅ Одобрить", callback_data=f"approve_{user_id}_{role}")
    builder.button(text="❌ Отклонить", callback_data=f"reject_{user_id}_{role}")
    return builder.as_markup()

@dp.message(Command("start"))
async def cmd_start(message: types.Message):
    user_data[message.from_user.id] = {}
    await message.answer(
        "👋 Привет! Ты хочешь подать заявку на роль в проекте FluxeTime?\n\nВыбери, на кого ты хочешь подать заявку:",
        reply_markup=get_roles_keyboard()
    )

@dp.callback_query(F.data.startswith("role_"))
async def process_role(callback: types.CallbackQuery):
    role = callback.data.split("_")[1]
    user_id = callback.from_user.id
    
    user_data[user_id]["role"] = role
    
    await callback.message.edit_text(
        f"🎯 Ты выбрал роль: **{role}**.\n\nТеперь ответь на несколько вопросов:\n\n1. Как тебя зовут?"
    )
    # Здесь можно было бы использовать FSM, но для простоты просто ждем следующее сообщение
    # Мы будем проверять состояние по наличию ключа в словаре

# Обработка ответов на вопросы (упрощенная логика через проверку состояния)
@dp.message()
async def handle_answers(message: types.Message):
    uid = message.from_user.id
    if uid not in user_data or "role" not in user_data[uid]:
        return # Игнорируем, если не идет процесс подачи заявки

    data = user_data[uid]

    # 1. Имя
    if "name" not in data:
        data["name"] = message.text
        await message.answer("2. Расскажи немного о своем опыте (или почему ты хочешь эту роль)?")
        return

    # 2. Опыт
    if "experience" not in data:
        data["experience"] = message.text
        await message.answer("3. Почему именно эта роль? Что ты можешь дать проекту?")
        return

    # 3. Причина
    if "reason" not in data:
        data["reason"] = message.text
        await message.answer("4. Скинь ссылку на свой профиль (Telegram, VK и т.д.) или напиши 'нет'.")
        return

    # 4. Профиль
    if "profile_link" not in data:
        data["profile_link"] = message.text
        
        # ФОРМИРОВАНИЕ ЗАЯВКИ
        text = (
            f"📩 **НОВАЯ ЗАЯВКА В FLUXETIME**\n\n"
            f"👤 **Пользователь:** @{message.from_user.username} ({message.from_user.first_name})\n"
            f"💼 **Роль:** {data['role']}\n"
            f"📝 **Имя:** {data['name']}\n"
            f"📜 **Опыт:** {data['experience']}\n"
            f"💡 **Мотивация:** {data['reason']}\n"
            f"🔗 **Профиль:** {data['profile_link']}"
        )
        
        try:
            await bot.send_message(
                chat_id=ADMIN_ID, 
                text=text, 
                parse_mode="Markdown",
                reply_markup=get_admin_keyboard(uid, data['role'])
            )
            await message.answer("✅ Твоя заявка успешно отправлена администрации! Жди решения.")
        except Exception as e:
            await message.answer("❌ Произошла ошибка при отправке заявки. Попробуй позже.")
        
        # Очищаем данные
        del user_data[uid]

@dp.callback_query(F.data.startswith("approve_"))
async def approve_application(callback: types.CallbackQuery):
    parts = callback.data.split("_")
    user_id = int(parts[1])
    role = parts[2]
    
    try:
        await bot.send_message(
            chat_id=user_id,
            text=f"🎉 Поздравляем! Твоя заявка на роль **{role}** была **ОДОБРЕНА**! Свяжись с администрацией для дальнейших инструкций."
        )
        await callback.answer("Заявка одобрена, пользователь уведомлен.")
    except:
        await callback.answer("Не удалось отправить сообщение пользователю (возможно, он заблокировал бота).")

@dp.callback_query(F.data.startswith("reject_"))
async def reject_application(callback: types.CallbackQuery):
    parts = callback.data.split("_")
    user_id = int(parts[1])
    role = parts[2]
    
    try:
        await bot.send_message(
            chat_id=user_id,
            text=f"😔 К сожалению, твоя заявка на роль **{role}** была **ОТКЛОНЕНА**. Не расстраивайся, пробуй снова позже!"
        )
        await callback.answer("Заявка отклонена, пользователь уведомлен.")
    except:
        await callback.answer("Не удалось отправить сообщение пользователю.")

async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
