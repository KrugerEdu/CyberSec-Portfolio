# Write-up: AWS CTF

**Categoria:** Web Exploitation / Forensics  
**Dificuldade:** Iniciante / Intermediário  
**Alvo Principal:** `https://nm04.bootupctf.net:8000`

---

## Resumo
Este write-up documenta a resolução de múltiplos desafios de um CTF, focados na exploração de uma aplicação web e em exercícios de forense digital. O processo envolveu enumeração de diretórios, análise de banco de dados vazados, quebra de hashes, inspeção de código-fonte, metadados de arquivos e análise de pacotes de rede (PCAP).

---

## 1. Web Exploitation (InvestiGate Portal)

### Portal 1: Enumerando Diretórios Ocultos
A exploração começou na URL `https://nm04.bootupctf.net:8000` utilizando o BurpSuite para interceptar e testar requisições. 
* Ao adicionar o caminho `/robots.txt` (um arquivo de instruções para bots e indexadores), fui redirecionado para uma página que continha código e uma nova URL.
* **Aprendizado:** É crucial testar caminhos e arquivos de configuração comuns durante a fase de reconhecimento, como `/robots.txt`, `/sitemap.xml`, `/.git` e `/.env`.

### Portal 2: Vazamento de Backup de Banco de Dados
* O redirecionamento encontrado no passo anterior apontava para `/tmpfiles/all`.
* Neste diretório, havia diversas opções, sendo que uma delas era um arquivo chamado `user.bak` (indicando um backup do servidor).
* Ao abrir o arquivo no VS Code, foi possível identificar a estrutura de um banco de dados SQLite3.
* Utilizando a ferramenta **DB Browser**, abri o arquivo e naveguei até a aba *Browse Data*.
* O banco de dados continha logins dos usuários `admin` e `dev`, e lá encontrei também a Flag 2.

### Portal 3: Quebra de Hash e Análise de Código Fonte
Os usuários encontrados no banco de dados possuíam senhas armazenadas em formato de *hash*:
* **Username:** `dev` | **Hash:** `18a7763dbf76f40177acbfda65e84214`
* **Username:** `admin` | **Hash:** `2365bfe9e7e5331dd2daf29d50bb0903`

Para quebrar a criptografia, inseri ambos os hashes em uma biblioteca de descriptografia, e apenas a hash do usuário `dev` deu *match*, revelando a senha: `samtheman`. 
* Com as credenciais em mãos, acessei uma página que supostamente estava em "desenvolvimento".
* Ao utilizar `Ctrl + Shift + I` para inspecionar o código da página, encontrei um comentário HTML: `"Not available for users yet, i think"`.
* Logo abaixo do comentário, havia dois links para o código-fonte da página de login, revelando a flag.
* **Flag 3:** `mne{r4ad1ng_s0urce_n3v3r_hurt5}`

### Portal 4: Chave de API
* Durante a exploração da aplicação, também foi descoberta uma chave de API exposta no sistema.
* **API Key:** `KQ7VJKY2YI5Z5RSN0U9NIURF22J8P63B`

---

## 2. Forense (Forensics)

### Data 1 - Exercício 1: Metadados Ocultos
* O arquivo fornecido era uma imagem. 
* Ao utilizar o comando `file`, revelei o tipo de arquivo real e descobri um comentário embutido na imagem contendo um código criptografado em hexadecimal.
* Após decodificar o valor hexadecimal, obtive a flag.
* **Flag:** `s0MeTAdaTa2210`

### VB - Exercício 3: Análise de Macros em Excel
* O arquivo alvo era um documento moderno do Microsoft Excel (`.xlsm` ou `.xlsb`).
* Ao listar seu conteúdo usando `ls`, identifiquei a estrutura típica de pastas de documentos compactados: `[Content_Types].xml`, `docProps`, `_rels` e `xl/`.
* Dentro do diretório `xl/`, encontrei o arquivo `vbaproject.bin`, que contém o código das macros VBA.
* Utilizando o comando `strings` no terminal Linux para ler o binário, observei várias funções corrompidas, incluindo o trecho `Functi@on AmI ected`.
* Como o objetivo era descobrir o nome da função que checava a conectividade, limpei o texto e deduzi a flag correta.
* **Flag:** `AmI Connected`

### Removed - Exercício 4: Redação (Censura) Falha
* Foi disponibilizado um arquivo `.zip` contendo um texto com tarjas de censura.
* Ao passar o mouse sobre as áreas censuradas e tentar selecionar o conteúdo como se fosse copiar, o texto oculto sob a tarja preta foi revelado de forma legível.
* **Flag:** `n1CeReDaCTION-sureLYNot911081`

### Epoch - Exercício 5: Análise de PCAP no Wireshark
* O desafio forneceu um arquivo `.pcap`, usado para gravar operações e tráfego de rede.
* Utilizando a ferramenta **Wireshark**, abri o arquivo para analisar as requisições gravadas.
* O objetivo era extrair o momento exato em que a primeira requisição de DNS foi feita, formatada no padrão Epoch.
* Fui até a aba `View` -> `Time Display Format` e mudei a visualização para `Seconds since 01 01 1970` para converter os registros de tempo.
* **Flag:** `1655931410`

---

## 3. Comandos Básicos (Cheatsheet)
Durante a resolução da máquina virtual Linux, utilizei os seguintes comandos para navegar e analisar os arquivos baixados:
* `cd ~/Downloads`: Muda o diretório (Change Directory) para a pasta de downloads.
* `ls`: Lista os arquivos visíveis do diretório.
* `ls -a`: Lista todos os arquivos, incluindo arquivos ocultos (hidden files).
* `ls -l`: Lista os arquivos em formato longo, mostrando o tamanho e permissões.
* `file ./program`: Verifica o tipo do arquivo (neste caso, de um arquivo chamado "program").
* `./program`: Executa o arquivo no diretório atual.