# NEXUS — Simulador de uma Cidade Inteligente

> Um simulador extensível de uma cidade virtual, desenvolvido em Python, no qual diferentes sistemas interagem para produzir um ambiente dinâmico.

## 1. Visão geral

O NEXUS é um projeto de simulação urbana. Seu objetivo é representar uma cidade em funcionamento, com infraestrutura, transportes, população, economia e acontecimentos que alteram o estado do ambiente.

A cidade deve operar continuamente, atualizando seus sistemas de forma coordenada. Os acontecimentos não devem ser previamente roteirizados: eles precisam surgir da combinação das regras implementadas.

Exemplo:

1. Uma tempestade começa.
2. A visibilidade diminui e determinadas vias ficam mais lentas.
3. Os ônibus passam a demorar mais para completar seus trajetos.
4. Os passageiros acumulam-se nas paradas.
5. Os atrasos afetam os horários dos passageiros e a eficiência do transporte.
6. O sistema registra as alterações e atualiza as estatísticas.

O objetivo é construir uma simulação coerente, modular e expansível, começando por um núcleo simples e adicionando complexidade gradualmente.

## 2. Objetivos técnicos

O projeto deverá exercitar:

- Programação Orientada a Objetos (POO).
- Classes, herança, composição e encapsulamento.
- Estruturas de dados e algoritmos.
- Grafos e algoritmos de busca de caminhos.
- Programação assíncrona com `asyncio`.
- Manipulação de arquivos JSON.
- Persistência com SQLite.
- Tratamento de exceções e logging.
- Testes automatizados.
- Análise de dados com pandas e visualização com matplotlib.
- Separação de responsabilidades e arquitetura modular.
- Construção de uma interface para visualizar a simulação.

Bibliotecas externas deverão ser adicionadas conforme as necessidades de cada fase, evitando dependências desnecessárias no núcleo.

## 3. Princípios fundamentais

### 3.1. A cidade é a fonte da verdade

A cidade deve manter o estado oficial da simulação. Posição dos veículos, estado das vias, localização dos passageiros e acontecimentos ativos devem ser consultados ou atualizados por meio dos sistemas responsáveis.

A interface visual não pode manter uma cópia independente da lógica da simulação.

### 3.2. Separação de responsabilidades

Cada classe deve ter uma responsabilidade clara.

Uma parada não deve calcular rotas de ônibus. Um ônibus não deve salvar diretamente toda a cidade em um arquivo. A interface não deve decidir quanto um veículo se desloca.

### 3.3. Estado consistente

As entidades não podem assumir estados impossíveis.

Exemplos:

- Um ônibus não pode estar em duas posições ao mesmo tempo.
- Um passageiro não pode estar simultaneamente dentro de dois veículos.
- Uma via bloqueada não pode ser utilizada por uma rota que proíba esse tipo de passagem.
- Um veículo não pode transportar mais passageiros do que sua capacidade.

### 3.4. Expansão gradual

Cada fase deve continuar funcionando após a implementação de novas funcionalidades. Alterações no sistema de transporte não devem exigir a reescrita do sistema econômico.

---

## 4. Arquitetura do projeto

A implementação deverá ser organizada em módulos. A estrutura abaixo é uma proposta de destino, não uma exigência para a primeira versão.

```text
nexus/
├── main.py
├── config.py
│
├── core/
│   ├── cidade.py
│   ├── simulacao.py
│   ├── relogio.py
│   └── eventos.py
│
├── entidades/
│   ├── entidade.py
│   ├── pessoa.py
│   ├── veiculo.py
│   ├── onibus.py
│   ├── parada.py
│   └── empresa.py
│
├── sistemas/
│   ├── transporte.py
│   ├── transito.py
│   ├── populacao.py
│   ├── economia.py
│   └── clima.py
│
├── infraestrutura/
│   ├── mapa.py
│   ├── via.py
│   └── rotas.py
│
├── persistencia/
│   ├── salvamento.py
│   └── banco.py
│
├── visualizacao/
│   ├── terminal.py
│   └── mapa_visual.py
│
├── dados/
│   └── cidades/
│
├── logs/
│
└── tests/
```

A estrutura poderá mudar quando surgirem necessidades reais. Não é necessário criar todos esses arquivos antecipadamente.

### Responsabilidades dos módulos

| Módulo | Responsabilidade |
|---|---|
| `core` | Coordenar o funcionamento da simulação e seu tempo |
| `entidades` | Representar objetos e seres presentes na cidade |
| `sistemas` | Implementar regras de funcionamento da cidade |
| `infraestrutura` | Representar o mapa, as vias e os caminhos |
| `persistencia` | Salvar e carregar os dados |
| `visualizacao` | Exibir o estado atual da simulação |
| `tests` | Verificar se as regras funcionam corretamente |

## 5. Modelo de entidades

### 5.1. Cidade

A classe `Cidade` representa o ambiente simulado.

Deve manter referências para:

- Infraestrutura e mapa.
- Entidades existentes.
- Sistemas ativos.
- Eventos em andamento.
- Configurações específicas da cidade.

A cidade também deve oferecer meios de registrar e remover entidades de maneira controlada.

### 5.2. Parada

Uma parada representa um local em que passageiros podem aguardar transporte.

Informações mínimas:

- Identificador único.
- Nome.
- Posição no mapa.
- Fila de passageiros.
- Informações sobre os veículos que atendem a parada, quando necessário.

Regras:

- Passageiros devem entrar na fila quando chegarem.
- Passageiros embarcados devem ser removidos da fila.
- O embarque deve respeitar a capacidade do veículo.
- Passageiros que não conseguirem embarcar devem continuar aguardando.

### 5.3. Veículo

A classe `Veiculo` representa o conceito geral de um veículo.

Deve definir as propriedades e comportamentos comuns, como:

- Identificador.
- Velocidade.
- Posição.
- Estado operacional.
- Capacidade, quando aplicável.

### 5.4. Ônibus

A classe `Onibus` especializa o conceito de veículo para o transporte coletivo.

Deve manter:

- Rota atribuída.
- Posição atual na rota.
- Velocidade atual.
- Lista de passageiros embarcados.
- Capacidade máxima.
- Próxima parada.
- Estado da viagem.

O ônibus deverá deslocar-se seguindo a rota, parar nos locais previstos, embarcar e desembarcar passageiros e respeitar os bloqueios de vias.

### 5.5. Pessoa

A classe `Pessoa` representa um indivíduo da população.

Na primeira versão, as pessoas podem ser apenas passageiros. Posteriormente, poderão possuir:

- Localização atual.
- Destino.
- Objetivo de deslocamento.
- Horários e compromissos.
- Dinheiro e renda.
- Emprego.
- Preferências de transporte.

A pessoa deve ter uma localização coerente: aguardando em uma parada, dentro de um veículo ou em outro local válido.

### 5.6. Empresa

Uma empresa representa uma organização econômica.

Em versões futuras, poderá possuir:

- Saldo.
- Funcionários.
- Receitas e despesas.
- Produtos ou serviços.
- Horários de funcionamento.
- Relações comerciais.

A economia não faz parte dos requisitos mínimos da primeira versão.

---

## 6. Infraestrutura e mapa

A cidade deverá ser representada por uma rede de locais conectados por vias.

### 6.1. Vias

Cada via deverá possuir, no mínimo:

- Identificador.
- Ponto de origem.
- Ponto de destino.
- Comprimento.
- Velocidade máxima.
- Estado de bloqueio.

Posteriormente, poderá ter faixas, sentido único, pedágios, limites variáveis e condições de tráfego.

### 6.2. Representação em grafo

O mapa deverá utilizar um grafo para representar as conexões entre os locais.

- Nós representam locais ou interseções.
- Arestas representam vias.
- Pesos representam custos de deslocamento, como tempo ou distância.

O grafo deverá permitir consultar os caminhos disponíveis entre dois locais.

### 6.3. Cálculo de rotas

O sistema de rotas deverá encontrar caminhos válidos entre origem e destino.

Na primeira versão, poderá utilizar distância como custo. Posteriormente, deverá considerar tempo estimado, congestionamentos e bloqueios.

A rota calculada deve ser independente da maneira como será desenhada na interface.

---

## 7. Sistema de transporte

O sistema de transporte coordena ônibus, paradas e rotas.

Suas responsabilidades incluem:

1. Registrar veículos e rotas.
2. Acompanhar o progresso das viagens.
3. Atualizar o deslocamento dos veículos.
4. Identificar a chegada às paradas.
5. Coordenar embarque e desembarque.
6. Detectar atrasos e interrupções.
7. Expor informações para relatórios e visualização.

O sistema de transporte não deve controlar diretamente a renderização gráfica.

### 7.1. Movimento

O movimento deverá considerar o tempo transcorrido.

Em vez de deslocar o ônibus uma distância fixa por atualização, o sistema deverá considerar a velocidade e o intervalo de tempo simulado.

Isso permite alterar a frequência de atualização sem mudar a velocidade física do veículo.

### 7.2. Paradas

Quando um ônibus chegar a uma parada:

1. O movimento deverá ser interrompido.
2. Os passageiros com destino àquela parada deverão desembarcar.
3. Passageiros aguardando poderão embarcar, respeitando a capacidade.
4. O sistema atualizará os estados dos passageiros.
5. O ônibus retomará a viagem quando estiver liberado para partir.

A duração do embarque e desembarque poderá ser introduzida em uma fase posterior.

---

## 8. Tempo e ciclo de simulação

O NEXUS deverá distinguir dois conceitos.

**Tempo real:** o tempo que efetivamente passa no computador.

**Tempo virtual:** o tempo que passa dentro da cidade simulada.

O relógio da simulação será responsável por avançar o tempo virtual. A velocidade da simulação poderá ser configurada, permitindo executá-la em tempo normal, acelerado ou pausado.

### 8.1. Ciclo principal

O ciclo deverá coordenar:

1. Avanço do relógio virtual.
2. Processamento de eventos pendentes.
3. Atualização dos sistemas.
4. Verificação da consistência do estado.
5. Registro de métricas.
6. Atualização da visualização.

A ordem deverá ser definida e documentada, especialmente quando dois sistemas alterarem dados relacionados.

### 8.2. Programação assíncrona

O projeto deverá utilizar `asyncio` para tarefas que precisem aguardar operações ou executar em intervalos diferentes.

Exemplos:

- Atualização de informações externas.
- Escrita periódica de relatórios.
- Interface de controle.
- Monitoramento da simulação.
- Tarefas de manutenção.

Não se deve criar uma tarefa assíncrona independente para cada objeto automaticamente. O projeto deverá evitar excesso de tarefas e condições de corrida.

A utilização de `asyncio` não torna cálculos pesados automaticamente paralelos. Se uma operação bloquear o event loop, deverá ser reestruturada ou executada fora dele quando apropriado.

### 8.3. Determinismo

Sempre que possível, uma simulação com os mesmos dados iniciais, configurações e sequência de eventos deverá produzir os mesmos resultados.

Para isso, o gerador aleatório deverá poder receber uma semente configurável, e a ordem de processamento deverá ser controlada.

---

## 9. Sistema de eventos

O sistema de eventos permitirá representar acontecimentos que modificam o ambiente.

Exemplos:

- Início de chuva.
- Acidente.
- Bloqueio de uma via.
- Falha mecânica.
- Aumento repentino da demanda.
- Encerramento de um bloqueio.

Um evento deverá especificar:

- Identificador.
- Tipo.
- Momento de início.
- Duração ou condição de encerramento.
- Alvo afetado.
- Efeitos aplicados.
- Estado atual.

Eventos temporários devem ter seus efeitos removidos quando terminarem. Um bloqueio não pode permanecer ativo indefinidamente por falta de encerramento.

Os sistemas responsáveis deverão aplicar os efeitos de cada evento, evitando que o sistema de eventos precise conhecer todos os detalhes internos da cidade.

---

## 10. População e comportamento

O sistema populacional deverá administrar a criação e o comportamento das pessoas.

Na primeira versão, as pessoas serão inseridas manualmente para testar o transporte.

Posteriormente, o sistema deverá permitir:

- Geração de indivíduos.
- Escolha de destinos.
- Planejamento de deslocamentos.
- Escolha de meios de transporte.
- Mudanças de comportamento diante de atrasos.
- Formação de padrões de demanda.

As decisões deverão seguir regras configuráveis. Não é necessário utilizar inteligência artificial ou aprendizado de máquina para produzir comportamentos interessantes.

---

## 11. Economia

O sistema econômico será responsável pelas movimentações financeiras da cidade.

Possíveis funcionalidades futuras:

- Salários.
- Receitas e despesas empresariais.
- Tarifas de transporte.
- Compra de produtos.
- Custos de operação.
- Falências.
- Indicadores econômicos.

Toda movimentação financeira deverá ter uma origem e um destino identificáveis.

O sistema não deverá criar ou destruir dinheiro silenciosamente. Toda criação ou remoção de saldo deverá corresponder a uma regra explícita, como emissão monetária, imposto ou despesa.

---

## 12. Dados, métricas e relatórios

A simulação deverá coletar dados para permitir a análise de seu comportamento.

Métricas sugeridas:

- Quantidade de veículos ativos.
- Passageiros transportados.
- Tempo médio de espera.
- Tempo médio de viagem.
- Atrasos acumulados.
- Ocupação dos veículos.
- Quantidade de vias bloqueadas.
- Receita e custo operacional, quando houver economia.

As métricas deverão ser registradas com referência ao tempo virtual e, quando necessário, ao tempo real.

O pandas poderá ser utilizado para analisar os dados e o matplotlib para gerar gráficos.

Os sistemas de análise não devem ser necessários para que a simulação básica funcione.

---

## 13. Persistência

O projeto deverá permitir salvar e restaurar o estado da cidade.

### Primeira etapa: JSON

O formato JSON poderá armazenar:

- Configuração da cidade.
- Entidades existentes.
- Estado dos veículos.
- Rotas.
- Posição dos passageiros.
- Tempo virtual.
- Eventos ativos.

### Etapa posterior: SQLite

O SQLite poderá armazenar dados históricos, registros de eventos, entidades persistentes e métricas.

A restauração deverá reconstruir as relações entre os objetos, validar os dados e verificar a consistência antes de retomar a execução.

Não se deve tentar serializar diretamente todos os objetos Python sem definir como suas relações serão reconstruídas.

---

## 14. Interface e visualização

A primeira interface será textual.

Ela deverá apresentar informações como:

- Tempo virtual.
- Posição dos ônibus.
- Próximas paradas.
- Quantidade de passageiros.
- Estado das vias.
- Eventos ativos.

Posteriormente, deverá existir uma visualização gráfica do mapa.

A interface gráfica deverá consultar o estado da simulação e solicitar comandos, como pausar, retomar, acelerar ou encerrar.

A renderização do mapa não deverá determinar as regras físicas ou econômicas da cidade.

---

## 15. Tratamento de erros e logs

O sistema deverá registrar acontecimentos relevantes, erros e operações importantes.

Exemplos:

- Veículo iniciou uma viagem.
- Ônibus chegou a uma parada.
- Evento foi ativado.
- Via foi bloqueada.
- Salvamento foi concluído.
- Uma entidade apresentou erro.

Erros recuperáveis deverão ser tratados sem encerrar toda a cidade quando isso for seguro.

Entretanto, o sistema não deverá esconder erros silenciosamente nem continuar após uma falha que comprometa a consistência do estado.

Mensagens de erro devem informar o contexto suficiente para facilitar a depuração.

---

## 16. Testes automatizados

Os testes deverão verificar as regras da simulação sem depender da interface gráfica.

Exemplos de testes:

- Um ônibus percorre corretamente uma rota.
- A posição do veículo respeita sua velocidade.
- Um passageiro não pode embarcar em um ônibus lotado.
- Passageiros desembarcam no destino correto.
- Uma via bloqueada não é utilizada por uma nova rota incompatível.
- Um evento temporário termina no momento esperado.
- O estado salvo pode ser carregado corretamente.
- Uma simulação com semente fixa produz resultados reproduzíveis.

Cada funcionalidade importante deverá ter testes que cubram seu comportamento normal e situações de erro.

---

## 17. Fases obrigatórias de desenvolvimento

### Fase 1 — Núcleo mínimo

Implementar:

- Classe `Cidade`.
- Classe `Parada`.
- Classe `Veiculo`.
- Classe `Onibus`.
- Rotas representadas inicialmente por uma sequência de paradas.
- Ciclo de atualização.
- Relatório textual.

**Critério de conclusão:** dois ônibus conseguem percorrer rotas definidas em uma cidade com três ou mais paradas, e suas posições são atualizadas corretamente.

### Fase 2 — Infraestrutura

Implementar:

- Mapa em grafo.
- Vias com comprimentos.
- Cálculo de distâncias.
- Busca de caminhos.
- Bloqueio de vias.
- Rotas calculadas automaticamente.

**Critério de conclusão:** o sistema consegue encontrar um caminho válido entre dois pontos e recalculá-lo quando uma via fica indisponível.

### Fase 3 — Transporte de passageiros

Implementar:

- Pessoas.
- Filas nas paradas.
- Capacidade dos veículos.
- Origem e destino dos passageiros.
- Embarque e desembarque.
- Métricas de espera e viagem.

**Critério de conclusão:** passageiros conseguem completar deslocamentos e as métricas correspondem aos acontecimentos da simulação.

### Fase 4 — Assincronismo e relógio

Implementar:

- Relógio virtual.
- Controle de velocidade.
- Pausa e retomada.
- Tarefas assíncronas.
- Encerramento controlado.
- Sincronização de operações compartilhadas quando necessária.

**Critério de conclusão:** os sistemas podem executar suas tarefas sem bloquear indevidamente o event loop ou deixar a cidade em um estado inconsistente.

### Fase 5 — Eventos e trânsito

Implementar:

- Congestionamentos.
- Chuva.
- Acidentes.
- Bloqueios temporários.
- Variação da velocidade.
- Replanejamento de rotas.

**Critério de conclusão:** eventos alteram efetivamente o comportamento dos veículos e deixam de produzir efeitos quando terminam.

### Fase 6 — População e economia

Implementar:

- Geração de passageiros.
- Escolha de destinos.
- Empresas.
- Receitas e despesas.
- Demanda variável.
- Relatórios operacionais e econômicos.

**Critério de conclusão:** as interações entre os sistemas produzem consequências mensuráveis e coerentes.

### Fase 7 — Persistência e confiabilidade

Implementar:

- Salvamento.
- Carregamento.
- SQLite.
- Logs.
- Testes automatizados.
- Validação dos dados.
- Tratamento de falhas.

**Critério de conclusão:** a cidade pode ser salva, restaurada e continuar funcionando com os dados consistentes.

### Fase 8 — Visualização e otimização

Implementar:

- Mapa gráfico.
- Animação dos veículos.
- Painel de métricas.
- Controles de simulação.
- Testes de desempenho.
- Otimização dos gargalos identificados.

**Critério de conclusão:** é possível observar e controlar a cidade visualmente sem misturar a interface com a lógica da simulação.

---

## 18. Requisitos mínimos da versão final

A versão final deverá:

- Possuir arquitetura modular.
- Representar uma cidade por meio de um grafo.
- Simular deslocamentos e passageiros.
- Possuir relógio virtual configurável.
- Utilizar tarefas assíncronas onde forem apropriadas.
- Possuir eventos que alterem o ambiente.
- Coletar e apresentar métricas.
- Salvar e restaurar o estado.
- Possuir testes automatizados.
- Oferecer uma interface de visualização.
- Manter as regras da simulação independentes da interface.

Funcionalidades econômicas avançadas e comportamentos complexos poderão ser expandidos depois que o núcleo estiver estável.

## 19. Regra de ouro do desenvolvimento

O NEXUS não deve ser construído de uma vez.

Cada fase deverá produzir uma versão funcional, testável e compreensível. A implementação deve começar simples e ganhar complexidade apenas quando existir uma necessidade concreta.

O objetivo não é escrever o máximo de código possível. É construir um sistema em que cada componente tenha uma função clara e todas as partes consigam trabalhar juntas.

**Primeiro faça a cidade funcionar. Depois faça a cidade parecer viva.**