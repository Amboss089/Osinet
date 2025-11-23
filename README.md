# Osinet
Osinet und Test bot
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes
import subprocess, os, glob

TELEGRAM_BOT_TOKEN = "8417722365:AAG3G0BhyniR47VHjVYa8ilYSVFN-L5I5b0"
CHAT_ID = "8272530372"
OUTPUT_DIR = "output"

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "OSINT Suite Bot aktiv!\nBefehle:\n/scan\n/latestpdf\n/status\n/help"
    )

async def scan(update: Update, context: ContextTypes.DEFAULT_TYPE):
    msg = await update.message.reply_text("Pipeline läuft...")
    subprocess.run(["python3", "pipeline.py"])
    
    # Sende PDF
    pdf_files = sorted(glob.glob(f"{OUTPUT_DIR}/*.pdf"))
    latest_pdf = pdf_files[-1]
    await context.bot.send_document(chat_id=CHAT_ID, document=open(latest_pdf, "rb"))
    
    await msg.edit_text(f"Pipeline abgeschlossen! PDF gesendet: {os.path.basename(latest_pdf)}")

async def latestpdf(update: Update, context: ContextTypes.DEFAULT_TYPE):
    pdf_files = sorted(glob.glob(f"{OUTPUT_DIR}/*.pdf"))
    if pdf_files:
        latest_pdf = pdf_files[-1]
        await context.bot.send_document(chat_id=CHAT_ID, document=open(latest_pdf, "rb"))
        await update.message.reply_text(f"Neueste PDF gesendet: {os.path.basename(latest_pdf)}")
    else:
        await update.message.reply_text("Keine PDF-Datei gefunden!")

async def status(update: Update, context: ContextTypes.DEFAULT_TYPE):
    pdf_files = sorted(glob.glob(f"{OUTPUT_DIR}/*.pdf"))
    if pdf_files:
        last_time = os.path.basename(pdf_files[-1])
        await update.message.reply_text(f"Pipeline idle.\nLetzte Analyse: {last_time}")
    else:
        await update.message.reply_text("Keine Analyse bisher durchgeführt.")

async def help_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "/start - Bot aktivieren\n"
        "/scan - Pipeline starten\n"
        "/latestpdf - Neueste PDF senden\n"
        "/status - Pipeline Status\n"
        "/help - Hilfe anzeigen"
    )

app = ApplicationBuilder().token(TELEGRAM_BOT_TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(CommandHandler("scan", scan))
app.add_handler(CommandHandler("latestpdf", latestpdf))
app.add_handler(CommandHandler("status", status))
app.add_handler(CommandHandler("help", help_cmd))

print("Bot läuft...")
app.run_polling()
