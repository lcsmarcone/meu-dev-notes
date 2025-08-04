# Comandos Básicos do Linux

Este arquivo contém uma lista de comandos essenciais do Linux, com exemplos práticos e anotações para consulta rápida.

---

## 🗂 Navegação entre Diretórios
| Comando         | Descrição                        | Exemplo                     |
|-----------------|----------------------------------|-----------------------------|
| `pwd`          | Mostra o diretório atual         | `pwd`                      |
| `cd`           | Navega entre diretórios          | `cd /home/usuario`         |
| `cd ..`        | Volta um nível                   | `cd ..`                    |
| `ls`           | Lista arquivos                   | `ls -lha`                  |

---

## 📄 Manipulação de Arquivos
| Comando              | Descrição                             | Exemplo                    |
|----------------------|---------------------------------------|----------------------------|
| `cp origem destino`  | Copia arquivos                        | `cp arquivo.txt /tmp/`    |
| `mv origem destino`  | Move ou renomeia arquivos             | `mv arquivo.txt backup/`  |
| `rm arquivo`         | Remove arquivo                       | `rm arquivo.txt`          |
| `rm -r pasta`        | Remove pasta recursivamente           | `rm -r old_folder/`       |
| `touch nome`         | Cria arquivo vazio                   | `touch novo.txt`          |

---

## 📦 Gerenciamento de Pacotes
### Arch/Manjaro
```bash
sudo pacman -Syu              # Atualiza pacotes
sudo pacman -S htop           # Instala o htop


## 🔑 Permissões de Arquivos
chmod 755 script.sh       # Permissão de leitura, escrita e execução para o dono
chmod +x script.sh        # Adiciona permissão de execução
chown usuario:grupo file  # Altera dono e grupo

## 🌐 Redes
ping google.com             # Testa conectividade
ifconfig                    # Mostra interfaces de rede (ou `ip a`)
netstat -tulnp              # Lista portas abertas
curl -O URL                 # Baixa arquivo da URL
wget URL                    # Baixa arquivo da URL


## 🖥️ Processos
ps aux                      # Lista processos em execução
top                         # Monitora processos em tempo real
htop                        # Versão interativa do top
kill PID                    # Encerra um processo pelo PID
killall nome-processo       # Encerra todos os processos com o mesmo nome

## 🛠️ Dicas Úteis
Use man comando para consultar o manual.

Combine comandos com pipes: cat arquivo | grep "texto".

Use history para listar comandos anteriores.

Atalho Ctrl + R busca comandos no histórico.