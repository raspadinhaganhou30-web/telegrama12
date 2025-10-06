import telegram
import schedule
import time
import random
import logging
import sys
from threading import Thread
from datetime import datetime, timedelta

# ================= CONFIGURAÇÃO =================
TOKEN = "7337195248:AAFyB2QAReKuaU_49A--LKP184XEkBtjdxI"
CHAT_ID = "-1003198115663"
TOPIC_ID = 7

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(message)s')
bot = telegram.Bot(token=TOKEN)

# ================= SEU BANCO DE DADOS =================
DOMINIOS = {
    "🥇 @sortuda.com": 200,
    "💎 @vip.com": 250, 
    "🚀 @ganhei.com": 300,
    "👑 @sorte.com": 350
}

PERIODOS = {
    "MANHÃ": ["08h-12h", "09h-13h"],
    "TARDE": ["12h-16h", "13h-17h", "14h-18h"],
    "NOITE": ["16h-20h", "17h-21h", "18h-22h"],
    "MADRUGADA": ["20h-24h", "21h-01h", "22h-02h"]
}

DICAS = [
    "Fluxo forte neste horário — quem entra agora costuma ver os melhores retornos!",
    "Sistema identificou padrão positivo — momento ideal para entrada!",
    "Horário de pico com retornos consistentes — não perca timing!",
    "Alta probabilidade de resultados positivos — estratégia validada!",
    "Momento estratégico do dia — retornos acima da média!",
    "Período com menor concorrência — oportunidade exclusiva!",
    "Análise técnica indica forte tendência de ganhos — entre agora!",
    "Janela de oportunidade limitada — ação recomendada!"
]

# ================= GERADOR DE SINAIS =================
def gerar_sinal(periodo):
    try:
        # Seleção aleatória baseada no período
        dominio, bonus = random.choice(list(DOMINIOS.items()))
        horario = random.choice(PERIODOS[periodo])
        confianca = random.randint(82, 94)
        dica = random.choice(DICAS)
        
        # Próximo horário baseado no período atual
        proximos_horarios = {
            "MANHÃ": "12:00",
            "TARDE": "18:00", 
            "NOITE": "22:00",
            "MADRUGADA": "08:00"
        }
        
        sinal = f"""🔮 SINAL DA {periodo}

🔥 DOMÍNIO: {dominio}
⏰ HORÁRIO: {horario}
💰 BÔNUS: {bonus}%
🎯 CONFIANÇA: {confianca}% ✅

💡 DICA: "{dica}"

🔄 PRÓXIMO SINAL: {proximos_horarios[periodo]} ⏳

https://raspadinhawin.org/"""
        
        return sinal
        
    except Exception as e:
        logging.error(f"Erro ao gerar sinal: {e}")
        return None

# ================= ENVIO DE SINAIS =================
def enviar_sinal(periodo):
    try:
        sinal = gerar_sinal(periodo)
        if sinal:
            bot.send_message(
                chat_id=CHAT_ID, 
                text=sinal,
                message_thread_id=TOPIC_ID
            )
            logging.info(f"Sinal {periodo} enviado com sucesso!")
        else:
            logging.error("Falha ao gerar sinal")
            
    except Exception as e:
        logging.error(f"Erro ao enviar sinal: {e}")

# ================= AGENDADOR =================
def agendar_sinais():
    # Sinais principais
    schedule.every().day.at("08:00").do(enviar_sinal, "MANHÃ")
    schedule.every().day.at("12:00").do(enviar_sinal, "TARDE")
    schedule.every().day.at("18:00").do(enviar_sinal, "NOITE")
    schedule.every().day.at("22:00").do(enviar_sinal, "MADRUGADA")
    
    logging.info("Agendador de sinais iniciado!")
    
    while True:
        schedule.run_pending()
        time.sleep(1)

# ================= COMANDOS DO BOT =================
def handle_messages():
    try:
        updates = bot.get_updates()
        for update in updates:
            if update.message and update.message.text:
                texto = update.message.text
                chat_id = update.message.chat_id
                
                if texto == "/sinal":
                    periodo = determinar_periodo_atual()
                    sinal = gerar_sinal(periodo)
                    if sinal:
                        bot.send_message(chat_id=chat_id, text=sinal)
                        
                elif texto == "/proximosinal":
                    proximo = obter_proximo_sinal()
                    bot.send_message(chat_id=chat_id, text=proximo)
                    
    except Exception as e:
        logging.error(f"Erro nos comandos: {e}")

def determinar_periodo_atual():
    hora = datetime.now().hour
    if 8 <= hora < 12: return "MANHÃ"
    elif 12 <= hora < 18: return "TARDE" 
    elif 18 <= hora < 22: return "NOITE"
    else: return "MADRUGADA"

def obter_proximo_sinal():
    hora = datetime.now().hour
    if hora < 8: return "🔄 PRÓXIMO SINAL: 08:00 ⏳"
    elif hora < 12: return "🔄 PRÓXIMO SINAL: 12:00 ⏳"
    elif hora < 18: return "🔄 PRÓXIMO SINAL: 18:00 ⏳"
    else: return "🔄 PRÓXIMO SINAL: 22:00 ⏳"

# ================= KEEP ALIVE =================
def keep_alive():
    while True:
        try:
            bot.get_me()
            time.sleep(300)
        except Exception as e:
            logging.error(f"Keep alive error: {e}")
            time.sleep(60)

# ================= INICIALIZAÇÃO =================
if __name__ == "__main__":
    logging.info("🤖 Bot de Sinais Iniciado!")
    
    # Iniciar threads
    Thread(target=agendar_sinais, daemon=True).start()
    Thread(target=keep_alive, daemon=True).start()
    
    # Manter principal ativo
    try:
        while True:
            handle_messages()
            time.sleep(10)
    except KeyboardInterrupt:
        logging.info("Bot encerrado pelo usuário")
