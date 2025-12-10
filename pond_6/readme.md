# Relatório Técnico de Análise de Segurança
## ESP32 Web Server - Controle de GPIOs

### 1. Introdução
Este relatório apresenta uma análise de segurança do código de um servidor web implementado em um ESP32 para controle de pinos GPIO. O objetivo é identificar vulnerabilidades presentes no código e descrever possíveis ataques que podem explorar essas falhas.

### 2. Descrição do Sistema
O código analisado implementa um servidor web simples que roda em um microcontrolador ESP32. O sistema permite que usuários controlem dois pinos GPIO (26 e 27) através de uma interface web, ligando e desligando esses pinos remotamente. O ESP32 se conecta a uma rede Wi-Fi e responde a requisições HTTP para alterar o estado dos pinos.

### 3. Análise de Vulnerabilidades
Após análise do código, foram identificadas as seguintes vulnerabilidades:

**3.1** Ausência de Autenticação
O servidor web não possui nenhum mecanismo de autenticação. Qualquer pessoa que tenha acesso à rede e conheça o endereço IP do dispositivo pode acessar e controlar os pinos GPIO.

**3.2** Falta de Criptografia (HTTP ao invés de HTTPS)
A comunicação entre o cliente e o servidor é feita através do protocolo HTTP, que não é criptografado. Isso significa que todas as informações trafegam em texto claro pela rede.

**3.3** Credenciais de Wi-Fi no Código
As credenciais da rede Wi-Fi (SSID e senha) estão escritas diretamente no código-fonte, o que facilita o vazamento dessas informações caso o código seja compartilhado ou acessado por pessoas não autorizadas.

**3.4** Falta de Validação de Entrada
O código não valida adequadamente as requisições recebidas, aceitando qualquer comando que corresponda aos padrões básicos definidos (como "/26/on" ou "/27/off").

**3.5** Consumo de Memória
A variável header acumula dados sem limite definido, o que pode causar problemas de memória se um atacante enviar requisições muito grandes.

### 4. Ataques Identificados

**Ataque 1: Acesso Não Autorizado aos Controles**

*Descrição do Ataque:*

Um atacante que esteja conectado à mesma rede Wi-Fi do ESP32 pode acessar a interface web e controlar os pinos GPIO sem qualquer tipo de verificação de identidade.

*Passo-a-passo do Ataque:*

- O atacante se conecta à mesma rede Wi-Fi que o ESP32
- O atacante descobre o endereço IP do ESP32 (pode fazer um scan da rede usando ferramentas simples)
- O atacante abre um navegador e digita o endereço IP do ESP32
- O atacante visualiza a interface web com os botões de controle
- O atacante clica nos botões ou acessa diretamente URLs como http://[IP_DO_ESP32]/26/on para controlar os pinos
- Os pinos GPIO são acionados sem qualquer verificação de autorização

*Probabilidade de Ocorrência:*

Alta (70-80%) - Este ataque é muito fácil de ser executado e requer apenas que o atacante esteja na mesma rede Wi-Fi. Em redes domésticas ou corporativas onde várias pessoas têm acesso, a probabilidade é bastante elevada. Não requer conhecimentos técnicos avançados.

*Impacto Estimado:*

Médio a Alto - O impacto depende do que está conectado aos pinos GPIO. Se forem dispositivos simples como LEDs, o impacto é baixo. Porém, se os pinos controlarem fechaduras, sistemas de alarme, ou equipamentos industriais, o impacto pode ser muito alto, causando problemas de segurança física, danos materiais ou interrupção de serviços.

*Risco Resultante:*

ALTO - Considerando a alta probabilidade de ocorrência combinada com um impacto potencialmente alto, este ataque representa um risco alto. A facilidade de execução e a ausência total de proteção tornam este cenário bastante preocupante.

[Acesse o vídeo de demonstração](https://youtube.com/shorts/E1-FKJS24k4) 

**Ataque 2: Ataque de Negação de Serviço (DoS)**
*Descrição do Ataque:*

Um atacante pode sobrecarregar o ESP32 enviando múltiplas requisições HTTP simultaneamente ou requisições muito grandes, fazendo com que o dispositivo pare de responder ou reinicie. Isso impede que usuários legítimos consigam controlar os pinos GPIO.

*Passo-a-passo do Ataque:*

- O atacante se conecta à mesma rede Wi-Fi que o ESP32
- O atacante descobre o endereço IP do ESP32
- O atacante usa um script simples ou ferramenta (como curl, wget, ou ferramentas de teste de carga) para enviar centenas de requisições HTTP simultâneas ao ESP32
- Alternativamente, o atacante pode enviar requisições HTTP com cabeçalhos muito grandes para consumir a memória limitada do ESP32
- O ESP32, com recursos de processamento e memória limitados, não consegue processar todas as requisições
- O dispositivo começa a travar, responder muito lentamente, ou reinicia completamente
- Durante o ataque, usuários legítimos não conseguem acessar a interface web nem controlar os pinos GPIO

*Probabilidade de Ocorrência:*

Média (40-50%) - Este ataque requer conhecimento técnico básico de como enviar requisições HTTP, mas as ferramentas necessárias são gratuitas e fáceis de usar. Scripts simples em Python ou até comandos de terminal podem executar este ataque. A probabilidade é média porque, embora não seja tão trivial quanto simplesmente acessar a página web, não exige conhecimentos avançados de programação ou hacking.

*Impacto Estimado:*

Alto - O impacto é significativo porque o ataque torna o sistema completamente indisponível. Se o ESP32 controla dispositivos críticos (como sistemas de segurança, alarmes, ou equipamentos industriais), a interrupção do serviço pode causar problemas sérios. Além disso, o dispositivo pode precisar ser reiniciado manualmente, causando tempo de inatividade. Em alguns casos, ataques DoS repetidos podem até danificar a memória flash do dispositivo devido a reinicializações constantes.

*Risco Resultante:*
ALTO - A combinação de probabilidade média-alta com impacto alto resulta em um risco alto. A facilidade relativa de execução do ataque, combinada com a capacidade de tornar o sistema completamente inoperante, representa uma ameaça séria. Especialmente considerando que o ESP32 possui recursos limitados e não tem proteções contra esse tipo de ataque implementadas no código.

### 5. Tabela Consolidada de Ataques

| Título do Ataque | Probabilidade | Impacto | Risco |
|------------------|---------------|---------|-------|
| Acesso Não Autorizado aos Controles | Alta (70-80%) | Médio a Alto | **ALTO** |
| Ataque de Negação de Serviço (DoS) | Média (40-50%) | Alto | **ALTO** |

### 6. Recomendações Básicas de Segurança
Para mitigar os riscos identificados, recomenda-se:

- Implementar autenticação: Adicionar um sistema de login com usuário e senha para impedir acesso não autorizado aos controles

- Usar HTTPS: Implementar comunicação criptografada para proteger os dados em trânsito

- Não incluir credenciais no código: Usar mecanismos de configuração separados para armazenar senhas, como arquivos de configuração ou EEPROM

- Limitar acesso por rede: Configurar firewall ou restrições de rede para limitar quem pode acessar o ESP32, permitindo apenas IPs conhecidos

- Implementar limitação de taxa (rate limiting): Estabelecer um limite de requisições por segundo ou por minuto para cada IP, impedindo que um atacante envie múltiplas requisições rapidamente

- Adicionar timeout e validação de tamanho: Limitar o tamanho máximo das requisições HTTP aceitas e implementar timeouts mais curtos para descartar conexões suspeitas

- Implementar watchdog timer: Configurar um mecanismo que reinicie o ESP32 automaticamente caso ele trave, garantindo recuperação rápida em caso de ataque

- Adicionar logs e monitoramento: Registrar todas as ações realizadas e implementar alertas para detectar padrões de ataque, como múltiplas requisições do mesmo IP


### 7. Conclusão
A análise revelou que o código possui vulnerabilidades críticas de segurança que expõem o sistema a diferentes tipos de ataques. Foram identificados dois ataques de alto risco: o "Acesso Não Autorizado aos Controles" e o "Ataque de Negação de Serviço (DoS)".

O primeiro ataque é extremamente fácil de executar e permite que qualquer pessoa na rede controle os dispositivos conectados aos pinos GPIO sem qualquer tipo de verificação. O segundo ataque pode tornar o sistema completamente indisponível, impedindo seu funcionamento normal e potencialmente causando danos aos dispositivos controlados. Ambos os ataques representam riscos altos devido à combinação de facilidade de execução e impacto significativo. O ESP32, sendo um dispositivo com recursos limitados de processamento e memória, é particularmente vulnerável a ataques DoS, tornando essencial a implementação de proteções adequadas.

É fundamental implementar as recomendações de segurança apresentadas antes de utilizar este sistema em qualquer ambiente real, seja doméstico, comercial ou industrial. A ausência de medidas básicas de segurança como autenticação, criptografia e proteção contra sobrecarga torna o sistema inadequado para uso em produção, especialmente se os pinos GPIO controlarem dispositivos críticos ou sensíveis. A implementação dessas melhorias não apenas protegerá o sistema contra os ataques identificados, mas também aumentará a confiabilidade e robustez geral da solução.