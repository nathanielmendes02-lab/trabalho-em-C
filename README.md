Sistema de Monitoramento de Temperatura



- Aluno: Nathaniel MENDES DE MORAES
- Disciplina: ENGENHARIA DE SOFTWARE
- Professora: Profa. Karla Sartin
- Título do projeto: Sistema de Monitoramento de Temperatura

Objetivo:

O programa simula um sistema de monitoramento de temperatura, como o de um
sensor industrial ou ambiental. Ele resolve o problema de identificar
automaticamente situações de risco — quando a temperatura permanece **acima
de um limite de segurança por várias leituras seguidas** — e de gerar um
relatório estatístico com o comportamento das leituras registradas
(média, maior valor, menor valor e percentual de leituras fora do limite).

Funcionamento do programa:

**Definição do limite de temperatura:** o próprio usuário digita o limite no
início da execução. O valor só é aceito se for numérico e estiver dentro de
uma faixa considerada plausível (-50.0 a 100.0); caso contrário, o programa
pede o valor novamente.

**Leitura das temperaturas:** após o limite ser definido, o programa entra em
um laço de monitoramento, pedindo uma temperatura por vez. A qualquer momento
o usuário pode digitar `-9999` para encerrar o monitoramento manualmente.

**Tratamento de valores inválidos:** cada leitura passa por duas verificações:

1. se o que foi digitado não é um número (`scanf` falha), o programa avisa e
   descarta a entrada, limpando o buffer para não travar a próxima leitura;
2. se é um número, mas está fora de uma faixa fisicamente aceitável (-50.0 a
   150.0), o programa também rejeita e pede a leitura novamente.

Nenhuma leitura inválida é contabilizada nas estatísticas.

**Identificação de temperaturas acima do limite:** toda leitura válida é
comparada ao limite configurado. Se for maior que o limite, ela é somada ao
total de leituras "acima do limite" (usado depois no percentual final) e
também incrementa o contador de consecutivas.

**Contagem de consecutivas:** existe um contador que soma 1 a cada leitura
acima do limite feita em sequência. Assim que aparece uma leitura **dentro**
do limite, esse contador volta a zero — ou seja, ele só conta sequências
ininterruptas de leituras perigosas.

**Condição de encerramento:** o monitoramento termina de duas formas:

- **Automática:** quando o contador de consecutivas atinge **3**, o programa
  emite um alerta e encerra sozinho, sem esperar mais entradas.
- **Manual:** quando o usuário digita o valor sentinela `-9999`.

Em ambos os casos, o programa exibe o relatório final antes de terminar.

Estruturas de repetição utilizadas:

O programa usa exclusivamente `while` e `do...while`, combinados conforme a
necessidade de cada trecho:

- **`do...while`** é usado nos três pontos em que uma leitura precisa ser
  *tentada* antes de poder ser *validada*: na leitura do limite de
  temperatura e na leitura de cada temperatura individual. Não há como saber
  se uma entrada é válida antes de pedi-la, então faz sentido executar o
  bloco pelo menos uma vez e só depois checar a condição.
- **`while`** é usado no laço principal do monitoramento, controlado pela
  variável `continuarMonitoramento`. Aqui a lógica é o oposto: antes de
  iniciar mais uma volta (pedir mais uma temperatura), o programa precisa
  checar se algum evento já decidiu encerrar o monitoramento (sentinela
  digitado ou alerta de 3 consecutivas atingido). Testar a condição **antes** de
  cada iteração evita pedir uma leitura extra desnecessária depois que o
  encerramento já foi decidido.

 Como executar:

 Windows (com MinGW-w64/gcc instalado)

```
gcc monitoramento.c -o monitoramento.exe
monitoramento.exe
```

> Caso o comando `gcc` não seja reconhecido, é porque o compilador ainda não
> está instalado/configurado no PATH. As alternativas mais simples são
> instalar o MinGW-w64 (ou o pacote `build-essential` via WSL) ou abrir o
> arquivo em uma IDE como Code::Blocks ou Dev-C++ e compilar por lá.

 Linux / macOS

```
gcc monitoramento.c -o monitoramento
./monitoramento
```

 Testes realizados:

 Teste 1 — Validação de entradas inválidas

Entrada de um limite não numérico (`abc`) e de um limite fora da faixa
(`500`) antes de um valor válido (`30`); em seguida, uma temperatura não
numérica (`xyz`) e uma fora da faixa (`999`) antes de leituras válidas.

![Teste 1](evidencias/teste01.png)

**Resultado:** todas as entradas inválidas foram rejeitadas com a mensagem
correspondente e nenhuma delas entrou no cálculo do relatório final (apenas
as 2 leituras válidas, 25 e 28, foram contabilizadas).

 Teste 2 — Temperaturas acima do limite, porém não consecutivas

Limite definido como `30`, com leituras `25, 35, 20, 40, 15` intercaladas
(sempre uma leitura dentro do limite entre as leituras acima dele),
encerrado manualmente com `-9999`.

![Teste 2](evidencias/teste02.png)

**Resultado:** as duas leituras acima do limite (35 e 40) foram detectadas
corretamente, mas o contador de consecutivas voltou a zero a cada leitura
dentro do limite — por isso o monitoramento **não** foi encerrado
automaticamente. O relatório mostra 2 de 5 leituras acima do limite (40%).

 Teste 3 — Três temperaturas consecutivas acima do limite

Limite definido como `30`, com três leituras seguidas acima dele (`32, 35,
40`), sem nenhuma leitura dentro do limite entre elas.

![Teste 3](evidencias/teste03.png)

**Resultado:** o contador de consecutivas chegou a 3 na terceira leitura, o
alerta foi disparado e o programa encerrou o monitoramento automaticamente,
sem esperar o valor sentinela. O relatório mostra 100% das leituras acima do
limite.

 Estruturas de repetição: por que `while` e `do...while` combinados?

Optei por combinar as duas estruturas porque elas resolvem problemas
diferentes dentro do mesmo programa. Sempre que o programa precisa **pedir
uma entrada ao usuário e só depois decidir se ela é válida**, o `do...while`
é a escolha natural: não existe forma de testar a condição antes, porque
ainda não há nenhum valor para testar — o corpo do laço (pedir e ler o
valor) precisa rodar pelo menos uma vez antes de qualquer verificação. É
exatamente o que acontece na leitura do limite e na leitura de cada
temperatura.

Já o laço principal do monitoramento usa `while` porque, ali, a diferença
entre testar antes ou depois da execução importa de verdade: depois de
processar uma leitura, o programa pode descobrir que deve parar (o usuário
digitou o valor sentinela, ou o contador de consecutivas chegou a 3). Se eu
usasse um `do...while` nesse ponto, o programa executaria mais uma iteração
completa — pedindo e lendo outra temperatura — mesmo já sabendo que deveria
ter parado. Usando `while`, a condição `continuarMonitoramento` é checada
**antes** de cada nova volta, então, assim que ela vira falsa, o laço para
imediatamente, sem pedir nenhuma leitura a mais. Foi justamente essa
necessidade — impedir uma iteração indesejada logo após o critério de
parada ser atingido — que tornou a diferença entre testar a condição antes
ou depois da execução relevante para a solução.
