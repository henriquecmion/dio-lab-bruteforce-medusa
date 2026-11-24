Simulação de Ataque de Força Bruta com Medusa no Kali Linux

Projeto prático desenvolvido para o Bootcamp Santander - Cibersegurança do Zero à Prática da DIO
.

Objetivo do Laboratório

Demonstrar na prática o funcionamento de um ataque de força bruta (Brute Force) direcionado a serviços de autenticação em ambiente controlado, utilizando a ferramenta Medusa no Kali Linux, compreendendo os riscos dessa vulnerabilidade e aplicando as devidas medidas defensivas e contramedidas.

Aviso Ético: Todos os testes foram executados exclusivamente em ambiente local e isolado (laboratório educacional), com alvos autorizados para fins de aprendizado e pesquisa em segurança ofensiva e defensiva.

Tecnologias e Ferramentas
Sistema Operacional Atacante: Kali Linux (Máquina Virtual)
Ferramenta de Auditoria: Medusa v2.2 (ou superior)
Sistema/Alvo de Testes: Ambiente Local de Laboratório (com serviço FTP ativo)
Protocolo Auditado: FTP (File Transfer Protocol - Porta 21)
Topologia do Laboratório
Máquina Atacante (Kali Linux): 192.168.15.45
Máquina Alvo (Metasploitable): 192.168.15.46
Modo de Conexão: Rede Local (Modo Bridge)
Passo a Passo da Execução
1. Mapeamento dos Endereços e Conectividade

Identificação do IP do Kali Linux e validação da conectividade com o alvo:

Identificação do IP no Kali Linux
Figura 1: Verificação da interface de rede eth0 com o IP local do Kali Linux (192.168.15.45).

Identificação do IP no Metasploitable
Figura 2: Verificação da interface de rede eth0 com o IP local do Metasploitable (192.168.15.46).

2. Teste de Conectividade (Ping)

Validação de rota direta e conectividade entre o Kali Linux e o Metasploitable:

bash
# Executado no Kali Linux testando a comunicação com o alvo
ping -c 3 192.168.15.46

Teste de Conectividade
Figura 3: Comunicação validada com 0% de perda de pacotes.

3. Preparação das Wordlists (Dicionários de Teste)

Para fins de demonstração no laboratório, foram criadas listas de credenciais contendo usuários comuns e senhas simuladas:

bash
# Criação da lista de usuários
echo -e "admin\nuser\nmsfadmin" > users.txt
# Criação da lista de senhas candidatas
echo -e "123456\npassword\nmsfadmin\nadmin" > passwords.txt
# Verificação do conteúdo criado
cat users.txt
cat passwords.txt

Criação das Wordlists
Figura 4: Listas de usuários e senhas preparadas para o teste controlado.

4. Execução do Teste com Medusa

O Medusa é uma ferramenta rápida, modular e paralela para auditoria de força bruta em múltiplos protocolos de rede.

Comando executado no terminal do Kali Linux:

bash
# Execução da ferramenta apontando para o módulo de FTP (Porta 21)
medusa -h 192.168.15.46 -U users.txt -P passwords.txt -M ftp -f
Parâmetros explicados:
-h 192.168.15.46: Especifica o endereço IP do host alvo.
-U users.txt: Caminho para a lista com os nomes de usuário a serem testados.
-P passwords.txt: Caminho para a lista com as senhas candidatas.
-M ftp: Carrega o módulo específico do protocolo FTP.
-f: Interrompe a execução imediatamente após encontrar a primeira credencial válida (stop on first success).

Execução do Medusa
Figura 5: Medusa testando as combinações e identificando com sucesso a credencial válida (msfadmin:msfadmin).

5. Análise de Logs no Alvo (Perspectiva Defensiva)

Ao inspecionar o arquivo de log do serviço FTP (/var/log/vsftpd.log) com privilégios administrativos no Metasploitable:

bash
sudo tail -n 20 /var/log/vsftpd.log

Observa-se claramente o padrão clássico de um ataque automatizado de força bruta originado a partir do IP do Kali Linux (192.168.15.45):

Múltiplas conexões (CONNECT) por segundo;
Tentativas falhadas sequenciais (FAIL LOGIN) com os usuários admin, user e msfadmin;
A tentativa bem-sucedida final (OK LOGIN) com o usuário msfadmin.

Logs de Autenticação no Alvo
Figura 6: Registro de auditoria do vsftpd no Metasploitable comprovando as tentativas em massa e o login obtido.

Medidas Defensivas e Contramedidas

A compreensão do vetor de ataque permite estruturar estratégias de defesa em profundidade:

Políticas de Bloqueio de Contas (Account Lockout):
Configuração de limite máximo de tentativas consecutivas incorretas (ex.: bloqueio temporário após 3 a 5 falhas).
Implementação de Fail2ban ou IPS:
Utilização de ferramentas como o fail2ban para analisar logs de autenticação e criar regras temporárias de firewall (iptables/nftables) bloqueando o IP de origem.
Autenticação Baseada em Chaves (SSH):
Desativar a autenticação por senha (PasswordAuthentication no no sshd_config) e exigir chaves criptográficas assimétricas (ed25519 ou RSA).
Autenticação Multifator (MFA/2FA):
Exigir uma segunda camada de validação temporal (TOTP), inviabilizando o acesso mesmo se a senha for descoberta.
Políticas de Complexidade e Senhas Fortes:
Obrigatoriedade de senhas longas (passphrases) para inviabilizar dicionários básicos e ataques de tamanho reduzido.
Conclusão

O laboratório evidenciou como serviços com autenticação fraca e exposta sem mitigação de taxa de requisições (rate limiting) são vulneráveis a ataques automatizados. A prática reforça que a proteção de qualquer serviço exposto exige defesas em camadas, monitoramento constante de logs e restrição de acesso por senhas puras.
