import requests
import smtplib
import time
from email.message import EmailMessage

# CONFIGURAÇÕES DE EMAIL
EMAIL_REMETENTE = '0000107043274xsp@al.educacao.sp.gov.br'
SENHA = 'zezinho230107'
EMAIL_DESTINATARIO = '0000107043274xsp@al.educacao.sp.gov.br'  # você mesmo receberá

# INTERVALO ENTRE VERIFICAÇÕES (em segundos)
INTERVALO = 60  # 1 minuto

# URL DA API
API_URL = 'https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd'

# ÚLTIMO VALOR INICIAL
ultimo_valor = None

def obter_valor_bitcoin():
    try:
        resposta = requests.get(API_URL)
        dados = resposta.json()
        return dados['bitcoin']['usd']
    except Exception as e:
        print("❌ Erro ao obter valor do Bitcoin:", e)
        return None

def enviar_email(valor):
    msg = EmailMessage()
    msg.set_content(f'O valor do Bitcoin mudou! Novo valor: ${valor}')
    msg['Subject'] = '🚨 Alerta: Bitcoin Alterado'
    msg['From'] = EMAIL_REMETENTE
    msg['To'] = EMAIL_DESTINATARIO

    try:
        with smtplib.SMTP_SSL('smtp.gmail.com', 465) as smtp:
            smtp.login(EMAIL_REMETENTE, SENHA)
            smtp.send_message(msg)
            print("✅ E-mail enviado com sucesso.")
    except Exception as e:
        print("❌ Erro ao enviar e-mail:", e)

def monitorar():
    global ultimo_valor
    while True:
        valor_atual = obter_valor_bitcoin()
        if valor_atual is not None:
            if ultimo_valor is None:
                ultimo_valor = valor_atual
                print(f"📈 Valor inicial: ${valor_atual}")
            elif valor_atual != ultimo_valor:
                print(f"⚠️ Alteração detectada! De ${ultimo_valor} para ${valor_atual}")
                enviar_email(valor_atual)
                ultimo_valor = valor_atual
            else:
                print(f"⏸ Sem alteração. Valor atual: ${valor_atual}")
        else:
            print("⚠️ Não foi possível obter o valor atual.")
        time.sleep(INTERVALO)

# Iniciar o monitoramento
monitorar()
