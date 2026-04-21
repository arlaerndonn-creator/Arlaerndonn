from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes
import asyncio

===== CONFIG =====

TOKEN = "8667656512:AAFlI4WC4WzyAQIm1TEFjiIsffPBKOtzJ6Y"
CHANNEL = "@AINZMETHOD"
BOT_USERNAME = "VIPREWARDBOT"
OWNER = "https://t.me/MAXROMEO7"
BUY_LINK = "https://t.me/COINBUYCHATBOT"

===== MEMORY DATABASE =====

users = {}

def get_user(uid):
if uid not in users:
users[uid] = {"coins": 0, "refs": 0, "ref_by": None, "vip": False}
return users[uid]

===== JOIN CHECK =====

async def is_joined(bot, user_id):
try:
member = await bot.get_chat_member(CHANNEL, user_id)
return member.status in ["member", "administrator", "creator"]
except:
return False

===== MENU =====

def menu():
return InlineKeyboardMarkup([
[InlineKeyboardButton("💰 𝗕𝗔𝗟𝗔𝗡𝗖𝗘", callback_data="balance"),
InlineKeyboardButton("👥 𝗥𝗘𝗙𝗘𝗥", callback_data="refer")],
[InlineKeyboardButton("🎁 𝗖𝗟𝗔𝗜𝗠", callback_data="claim"),
InlineKeyboardButton("📋 𝗧𝗔𝗦𝗞", callback_data="task")],
[InlineKeyboardButton("🛒 𝗕𝗨𝗬 𝗖𝗢𝗜𝗡", callback_data="buy")]
])

===== LOADING =====

async def loading(q, text):
bars = ["░░░░░░░░░░","█░░░░░░░░░","██░░░░░░░░","███░░░░░░░",
"████░░░░░░","█████░░░░░","██████░░░░",
"███████░░░","████████░░","█████████░","██████████"]

for b in bars:
try:
await q.edit_message_text(f"⚡ 𝗟𝗢𝗔𝗗𝗜𝗡𝗚\n[{b}]\n{text}")
await asyncio.sleep(0.12)
except:
pass

===== COIN BAR =====

def coin_bar(c):
filled = min(c // 10, 10)
return "🟩"filled + "⬛"(10-filled)

===== START =====

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
user = update.effective_user
uid = user.id

JOIN LOCK

if not await is_joined(context.bot, uid):
return await update.message.reply_text(
"🔒 𝗔𝗖𝗖𝗘𝗦𝗦 𝗟𝗢𝗖𝗞𝗘𝗗\n\nJOIN CHANNEL FIRST",
reply_markup=InlineKeyboardMarkup([
[InlineKeyboardButton("📢 JOIN", url=f"https://t.me/{CHANNEL.replace('@','')}")],
[InlineKeyboardButton("✅ VERIFY", callback_data="check")]
])
)

u = get_user(uid)

REF SYSTEM

if context.args:
ref = context.args[0]
if ref.isdigit() and u["ref_by"] is None and ref != str(uid):
u["ref_by"] = ref
ref_user = get_user(int(ref))
reward = 10 if ref_user["vip"] else 5
ref_user["coins"] += reward
ref_user["refs"] += 1

await update.message.reply_text(f"""

╔══════════════════════════╗
👑 𝗩𝗜𝗣 𝗥𝗘𝗪𝗔𝗥𝗗 👑
╚══════════════════════════╝

✨ 𝗪𝗘𝗟𝗖𝗢𝗠𝗘 @{(user.username or "USER").upper()}

💎 𝗘𝗔𝗥𝗡 • 𝗖𝗟𝗔𝗜𝗠 • 𝗚𝗥𝗢𝗪
⚡ 𝗨𝗟𝗧𝗥𝗔 𝗣𝗥𝗘𝗠𝗜𝗨𝗠 𝗣𝗔𝗡𝗘𝗟

━━━━━━━━━━━━━━━━━━
🔥 OWNER: @MAXROMEO7
━━━━━━━━━━━━━━━━━━
""", reply_markup=menu())

===== BUTTON HANDLER =====

async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
q = update.callback_query
await q.answer()

uid = q.from_user.id
user = get_user(uid)

VERIFY BUTTON

if q.data == "check":
if not await is_joined(context.bot, uid):
return await q.answer("❌ JOIN FIRST", show_alert=True)
return await q.edit_message_text("🔓 ACCESS UNLOCKED", reply_markup=menu())

BALANCE

if q.data == "balance":
await loading(q, "FETCHING PROFILE")

return await q.edit_message_text(f"""

╔══════════════════════╗
👤 𝗣𝗥𝗢𝗙𝗜𝗟𝗘
╚══════════════════════╝

👤 @{(q.from_user.username or "USER").upper()}
🏷 {"👑 VIP" if user["vip"] else "⚪ NORMAL"}

━━━━━━━━━━━━━━━━━━
💰 {user['coins']}
{coin_bar(user['coins'])}

👥 {user['refs']}
━━━━━━━━━━━━━━━━━━
""", reply_markup=menu())

REFER

if q.data == "refer":
await loading(q, "GENERATING LINK")

link = f"https://t.me/{BOT_USERNAME}?start={uid}"      

return await q.edit_message_text(f"""

╔══════════════════════╗
👥 𝗥𝗘𝗙𝗘𝗥
╚══════════════════════╝

🔗 {link}

💰 +5 PER USER
👑 +10 VIP
""", reply_markup=menu())

CLAIM MENU

if q.data == "claim":
await loading(q, "OPENING PANEL")

return await q.edit_message_text("""

╔══════════════════════╗
🎁 𝗖𝗟𝗔𝗜𝗠
╚══════════════════════╝

SELECT REWARD
""", reply_markup=InlineKeyboardMarkup([
[InlineKeyboardButton("💳 CC (100)", callback_data="wcc")],
[InlineKeyboardButton("💎 FF (200)", callback_data="wff")],
[InlineKeyboardButton("🎮 UC (10)", callback_data="wbgmi")],
[InlineKeyboardButton("⬅ BACK", callback_data="back")]
]))

WITHDRAW

if q.data in ["wcc","wff","wbgmi"]:
await loading(q, "PROCESSING")

cost = {"wcc":100,"wff":200,"wbgmi":10}[q.data]      
name = {"wcc":"CC","wff":"FF DIAMOND","wbgmi":"UC"}[q.data]      

if user["coins"] < cost:      
    return await q.edit_message_text("❌ NOT ENOUGH COINS", reply_markup=menu())      

user["coins"] -= cost      

return await q.edit_message_text(f"""

╔══════════════════════╗
✅ SUCCESS
╚══════════════════════╝

🎁 {name}
💰 -{cost} COINS

📩 CONTACT OWNER
""", reply_markup=InlineKeyboardMarkup([
[InlineKeyboardButton("📩 CONTACT", url=OWNER)],
[InlineKeyboardButton("⬅ BACK", callback_data="back")]
]))

TASK

if q.data == "task":
await loading(q, "LOADING TASKS")

return await q.edit_message_text("""

📋 TASK LIST

1. PROMOTE CHANNEL


2. WORK FOR OWNER


3. BOT TASK
""", reply_markup=InlineKeyboardMarkup([
[InlineKeyboardButton("📩 CONTACT", url=OWNER)],
[InlineKeyboardButton("⬅ BACK", callback_data="back")]
]))



BUY

if q.data == "buy":
await loading(q, "OPENING STORE")

return await q.edit_message_text("""

🛒 BUY COINS

💰 1 COIN = ₹1
⚡ INSTANT DELIVERY
""", reply_markup=InlineKeyboardMarkup([
[InlineKeyboardButton("💳 BUY NOW", url=BUY_LINK)],
[InlineKeyboardButton("⬅ BACK", callback_data="back")]
]))

BACK

if q.data == "back":
return await q.edit_message_text("🏠 MAIN MENU", reply_markup=menu())

===== RUN BOT =====

app = ApplicationBuilder().token(TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(CallbackQueryHandler(button))

print("🚀 BOT RUNNING...")
app.run_polling()
