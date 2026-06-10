import telebot
from telebot import types
from telebot.types import MessageEntity
import random
import string
import emoji
import json
import os

# --- CONFIGURATION ---
TOKEN = "7964249283:AAFCE4WGN-zXx9pA9UzZkFgCJSRRGICXmuY" 
ADMIN_ID = 5127370488
bot = telebot.TeleBot(TOKEN)

# Force Join Channels
MUST_JOIN = ["@OBITO_AURAFARMER" ]

# Database files
USERS_FILE = "users.json"
POSTS_FILE = "posts.json"

# Load Users
if os.path.exists(USERS_FILE):
    with open(USERS_FILE, "r") as f:
        registered_users = set(json.load(f))
else:
    registered_users = set()

def save_users():
    with open(USERS_FILE, "w") as f:
        json.dump(list(registered_users), f)

# Premium Emoji IDs
PREMIUM_EMOJIS = [
    "6235355429237430006", "6147815573314082674", "5350427505805238170", 
    "5287267357427776826", "5222447122586036397", "5224180824789770658", 
    "5224663892646452625", "5224205542326557875", "5221953158397321906", 
    "5309981979167463973", "5309928798882395910", "5246765089977037900", 
    "5285161474833006232", "5285078504654783223", "5426918974971486256", 
    "5474143948572223102", "5472057595193743789", "5472159355853888315", 
    "6307665627481903641", "6088957586302831521", "6109328624777694916", 
    "6109693533789096849", "6109213820301872263", "6109557847182281178", 
    "6109447084270684884", "6109281659310312426", "6111423933162981989", 
    "6109211870386720327", "6109655025112320594", "6123114099703287427", 
    "6122990988760715630", "6123066743393881068", "6120791828066208322", 
    "6221756527691173256", "6168137610507062619", "6192627406654671561", 
    "6190651597144461028", "6192895915125116350", "6192532968913767492", 
    "6217491333108470219", "5463071033256848094", "6235403472741603087", 
    "6147565374289220368", "6147464060305676048", "6147524086768604985", 
    "5449449325434266744", "6273840152980755328", "6276057176444246654", 
    "6273997026661241933", "6273726078649372769", "6274007313107915274", 
    "5978776771623914876", "5978686323907628843", "5852873584912896283", 
    "5895297528106061174", "5895735846698487922", "5895343514320899727", 
    "5913754823643107921", "5197434882321567830", "5463071033256848094", 
    "5463256910851546817", "5463423955014529788", "5465443379917629504", 
    "5465465194056525619", "6235620067942341623", "6235717714023814969", 
    "6235593671073339928", "6147617184479711380", "6235403472741603087", 
    "5346181118884331907", "5971944878815317190",
    "6132184924603554220", "6237519835056575931", "6086702706997597042",
    "6089117655438990048", "6129711392808247546", "6129732880529628243",
    "6120464813551260125", "6120726896750629340", "5312361253610475399",
    "6104631352190043951", "6123129707614441341", "5309928798882395910",
    "6237825705447527988"
]

posts_db = {}
user_channels = {}
temp_data = {}

# --- UTILS ---
def generate_id():
    return ''.join(random.choices(string.ascii_uppercase + string.digits, k=6))

def check_must_join(user_id):
    for channel in MUST_JOIN:
        try:
            member = bot.get_chat_member(channel, user_id)
            if member.status in ['left', 'kicked']:
                return False
        except:
            return False
    return True

def process_text_and_entities(message):
    input_text = message.text or message.caption or ""
    original_entities = message.entities or message.caption_entities or []
    final_text, new_entities = "", []
    offset_map = {}
    current_old_offset, current_new_offset = 0, 0
    
    for char in input_text:
        offset_map[current_old_offset] = current_new_offset
        if emoji.is_emoji(char):
            rand_id = random.choice(PREMIUM_EMOJIS)
            placeholder = "✨" 
            new_entities.append(MessageEntity(type="custom_emoji", offset=current_new_offset, length=len(placeholder), custom_emoji_id=rand_id))
            final_text += placeholder
            char_len = len(char.encode('utf-16-le')) // 2
            current_old_offset += char_len
            current_new_offset += len(placeholder)
        else:
            final_text += char
            current_old_offset += 1
            current_new_offset += 1
            
    offset_map[current_old_offset] = current_new_offset
    for ent in original_entities:
        if ent.type == "custom_emoji": continue
        new_start, new_end = offset_map.get(ent.offset), offset_map.get(ent.offset + ent.length)
        if new_start is not None and new_end is not None:
            new_entities.append(MessageEntity(type=ent.type, offset=new_start, length=new_end - new_start, url=ent.url, user=ent.user, language=ent.language, custom_emoji_id=ent.custom_emoji_id))
    return final_text, new_entities

# --- KEYBOARDS ---
def main_menu(user_id):
    markup = types.ReplyKeyboardMarkup(row_width=2, resize_keyboard=True)
    markup.add("MAKE POST 🏣", "CONNECT CHANNELS 💎", "SEND TO CHANNEL 💎")
    # Yahan check ho raha hai ki user admin hai ya nahi
    if user_id == ADMIN_ID:
        markup.add("ADMIN PANEL 🛠")
    return markup

def admin_keyboard():
    markup = types.InlineKeyboardMarkup()
    markup.row(types.InlineKeyboardButton("STATUS 📊", callback_data="admin_status"),
               types.InlineKeyboardButton("BROADCAST 📢", callback_data="admin_broadcast"))
    return markup

# --- HANDLERS ---
@bot.message_handler(commands=['start'])
def welcome(message):
    if message.chat.id not in registered_users:
        registered_users.add(message.chat.id)
        save_users()
    
    if not check_must_join(message.from_user.id):
        markup = types.InlineKeyboardMarkup()
        markup.row(types.InlineKeyboardButton("JOIN CHANNEL", url="https://t.me/ANSH_GANG"))
        markup.row(types.InlineKeyboardButton("JOIN CHANNEL", url="https://t.me/VIPKING123454"))
        markup.row(types.InlineKeyboardButton("JOIN CHANNEL", url="https://t.me/ANSH_LIKE"))
        markup.row(types.InlineKeyboardButton("VERIFY JOIN ✅", callback_data="check_join"))
        return bot.send_message(message.chat.id, "<b>❌ ACCESS DENIED!</b>\n\nTO USE THE BOT JOIN ALL THE CHANNELS AND CLICK ON THE VERIFY BUTTON BELOW", reply_markup=markup, parse_mode="HTML")
    
    # User ID pass kiya taaki admin button ka logic chale
    bot.send_message(message.chat.id, "<b>WELCOME TO NARUTO CODEX PREMIUM EMOJI BOT!</b>\n\nTHIS BOT CONVERT YOUR NORMAL EMOJI INTO PREMIUM EMOJI", reply_markup=main_menu(message.chat.id), parse_mode="HTML")

@bot.callback_query_handler(func=lambda call: call.data == "check_join")
def verify_callback(call):
    if check_must_join(call.from_user.id):
        bot.answer_callback_query(call.id, "Success! Ab aap bot use kar sakte hain।")
        bot.send_message(call.message.chat.id, "✅ Verified! /start likho shuru karne ke liye।", reply_markup=main_menu(call.message.chat.id))
    else:
        bot.answer_callback_query(call.id, "❌ HAVEN'T JOINED YET", show_alert=True)

# --- ADMIN PANEL FEATURES ---
@bot.message_handler(func=lambda m: m.text == "ADMIN PANEL 🛠" and m.chat.id == ADMIN_ID)
def admin_panel(message):
    bot.send_message(message.chat.id, "<b> ANSH KING ADMIN PANEL</b>\n\nWelcome", reply_markup=admin_keyboard(), parse_mode="HTML")

@bot.callback_query_handler(func=lambda call: call.data.startswith("admin_"))
def admin_callback(call):
    if call.data == "admin_status":
        total = len(registered_users)
        bot.answer_callback_query(call.id)
        bot.send_message(call.message.chat.id, f"<b>📊 BOT STATUS</b>\n\nTotal Users: <code>{total}</code>", parse_mode="HTML")
    
    elif call.data == "admin_broadcast":
        bot.answer_callback_query(call.id)
        sent = bot.send_message(call.message.chat.id, "Broadcast message bhejo (Text/Photo/Video sab chalega):")
        bot.register_next_step_handler(sent, perform_broadcast)

def perform_broadcast(message):
    success, fail = 0, 0
    for user_id in registered_users:
        try:
            bot.copy_message(user_id, message.chat.id, message.message_id)
            success += 1
        except:
            fail += 1
    bot.send_message(ADMIN_ID, f"<b>📢 BROADCAST COMPLETE</b>\n\n✅ Success: {success}\n❌ Failed: {fail}", parse_mode="HTML")

# --- MAKE POST LOGIC (SAME AS BEFORE) ---
@bot.message_handler(func=lambda m: m.text == "MAKE POST 🏣")
def start_post(message):
    sent = bot.send_message(message.chat.id, "SEND YOUR POST WITH NORMAL EMOJIS:")
    bot.register_next_step_handler(sent, process_post_content)

def process_post_content(message):
    photo_id = message.photo[-1].file_id if message.content_type == 'photo' else None
    processed_text, entities = process_text_and_entities(message)
    temp_data[message.chat.id] = {"id": generate_id(), "content": processed_text, "entities": entities, "photo_id": photo_id}
    sent = bot.send_message(message.chat.id, "SEND THE BUTTON NAME:")
    bot.register_next_step_handler(sent, process_button_name)

def process_button_name(message):
    temp_data[message.chat.id]["btn_name"] = message.text
    sent = bot.send_message(message.chat.id, "SEND THE BUTTON LINK (URL):")
    bot.register_next_step_handler(sent, process_button_link)

def process_button_link(message):
    user_id = message.chat.id
    data = temp_data[user_id]
    data["btn_url"] = message.text
    posts_db[data["id"]] = data
    markup = types.InlineKeyboardMarkup().add(types.InlineKeyboardButton(text=data["btn_name"], url=data["btn_url"]))
    bot.send_message(user_id, f"✅ Done! ID: <code>{data['id']}</code>", parse_mode="HTML")
    if data["photo_id"]:
        bot.send_photo(user_id, data["photo_id"], caption=data["content"], caption_entities=data["entities"], reply_markup=markup)
    else:
        bot.send_message(user_id, data["content"], entities=data["entities"], reply_markup=markup)

# --- CHANNEL & SEND LOGIC (SAME AS BEFORE) ---
@bot.message_handler(func=lambda m: m.text == "CONNECT CHANNELS 💎")
def ask_channel(message):
    sent = bot.send_message(message.chat.id, "FORWARD A MESSAGE FROM THE CHANNEL OR WRITE @userbame:", parse_mode="HTML")
    bot.register_next_step_handler(sent, verify_channel)

def verify_channel(message):
    chat_id = message.forward_from_chat.id if message.forward_from_chat else message.text
    try:
        member = bot.get_chat_member(chat_id, bot.get_me().id)
        if member.status in ['administrator', 'creator']:
            chat_info = bot.get_chat(chat_id)
            if message.chat.id not in user_channels: user_channels[message.chat.id] = []
            user_channels[message.chat.id].append({"id": chat_info.id, "title": chat_info.title})
            bot.send_message(message.chat.id, f"✅ Connected: {chat_info.title}")
        else:
            bot.send_message(message.chat.id, "❌ Bot ko admin banao pehle।")
    except:
        bot.send_message(message.chat.id, "❌ CHANNEL NOT FOUND")

@bot.message_handler(func=lambda m: m.text == "SEND TO CHANNEL 💎")
def ask_post_id(message):
    sent = bot.send_message(message.chat.id, "Enter Post ID:")
    bot.register_next_step_handler(sent, select_channel)

def select_channel(message):
    post_id = message.text.upper()
    if post_id not in posts_db or message.chat.id not in user_channels:
        bot.send_message(message.chat.id, "❌ ID galat hai ya channel connect nahi hai।")
        return
    markup = types.InlineKeyboardMarkup()
    for chan in user_channels[message.chat.id]:
        markup.add(types.InlineKeyboardButton(text=chan["title"], callback_data=f"send_{post_id}_{chan['id']}"))
    bot.send_message(message.chat.id, "Konsa channel?", reply_markup=markup)

@bot.callback_query_handler(func=lambda call: call.data.startswith("send_"))
def perform_send(call):
    _, pid, cid = call.data.split("_")
    data = posts_db.get(pid)
    if data:
        markup = types.InlineKeyboardMarkup().add(types.InlineKeyboardButton(text=data["btn_name"], url=data["btn_url"]))
        try:
            if data["photo_id"]: bot.send_photo(cid, data["photo_id"], caption=data["content"], caption_entities=data["entities"], reply_markup=markup)
            else: bot.send_message(cid, data["content"], entities=data["entities"], reply_markup=markup)
            bot.edit_message_text("✅ Sent successfully!", call.message.chat.id, call.message.message_id)
        except Exception as e:
            bot.answer_callback_query(call.id, f"Error: {e}", show_alert=True)

print("Naruto Codex Bot is active with Admin Panel...")
bot.infinity_polling()