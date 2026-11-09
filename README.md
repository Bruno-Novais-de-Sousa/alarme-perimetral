# Sistema de Segurança Patrimonial com Arduino

Protótipo funcional de **segurança patrimonial automatizada** baseado na plataforma Arduino, com foco no **monitoramento de perímetro em tempo real**. O sistema detecta invasões por meio de barreiras ópticas a laser e sensores de presença e responde de forma imediata com alertas sonoros e acionamento de atuadores de segurança.

---

## Sumário

- [Visão geral](#visão-geral)
- [Funcionalidades](#funcionalidades)
- [Arquitetura do sistema](#arquitetura-do-sistema)
- [Componentes utilizados](#componentes-utilizados)
- [Esquema de ligação](#esquema-de-ligação)
- [Lógica de funcionamento](#lógica-de-funcionamento)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Como executar](#como-executar)
- [Testes e calibração](#testes-e-calibração)
- [Limitações e melhorias futuras](#limitações-e-melhorias-futuras)
- [Tecnologias](#tecnologias)
- [Autores](#autores)
- [Licença](#licença)

---

## Visão geral

O projeto consiste no desenvolvimento de um sistema de alarme de perímetro de baixo custo, estruturado em torno de um microcontrolador Arduino. O **firmware, escrito em C/C++**, utiliza **lógica condicional estruturada** e **tratamento de eventos** para gerenciar os diferentes estados do alarme (desarmado, armado, disparado).

A detecção de violação do perímetro é feita pela combinação de dois tipos de sensores:

- **Barreiras ópticas com diodo laser + sensor receptor (LDR):** o rompimento do feixe indica uma tentativa de passagem.
- **Sensores de presença (PIR):** identificam movimento na área monitorada.

Ao detectar uma violação, o software processa os sinais dos sensores e coordena, de forma **sincronizada**, o acionamento de **alertas sonoros** e **atuadores de segurança**.

## Funcionalidades

- Monitoramento contínuo do perímetro em tempo real
- Detecção por barreira óptica a laser (quebra de feixe)
- Detecção de movimento por sensor de presença
- Máquina de estados do alarme (desarmado / armado / disparado)
- Acionamento sincronizado de buzzer/sirene e atuadores
- Tratamento de eventos para resposta imediata à invasão
- Indicação visual do estado do sistema por LEDs
- Código modular e comentado, fácil de adaptar

## Arquitetura do sistema

```
┌──────────────────┐     ┌──────────────────┐
│ Barreira óptica  │     │ Sensor de        │
│ (Laser + LDR)    │     │ presença (PIR)   │
└────────┬─────────┘     └────────┬─────────┘
         │                        │
         └──────────┬─────────────┘
                    ▼
          ┌───────────────────┐
          │      ARDUINO      │
          │  Firmware C/C++   │
          │  • Leitura        │
          │  • Lógica de      │
          │    estados        │
          │  • Eventos        │
          └─────────┬─────────┘
                    │
         ┌──────────┼──────────────┐
         ▼          ▼              ▼
   ┌──────────┐ ┌──────────┐ ┌────────────┐
   │ Buzzer / │ │   LEDs   │ │ Atuadores  │
   │  Sirene  │ │ (status) │ │ (relé etc.)│
   └──────────┘ └──────────┘ └────────────┘
```

## Componentes utilizados

> Ajuste a lista conforme o que foi efetivamente usado na montagem.

| Qtd. | Componente | Função |
|:----:|------------|--------|
| 1 | Arduino (Uno ou compatível) | Unidade de processamento |
| 1+ | Módulo diodo laser | Emissor da barreira óptica |
| 1+ | LDR (ou módulo receptor de luz) | Receptor da barreira óptica |
| 1+ | Sensor de presença PIR (ex.: HC-SR501) | Detecção de movimento |
| 1 | Buzzer / sirene | Alerta sonoro |
| 1+ | Atuador de segurança (relé, servo, trava etc.) | Resposta física à invasão |
| 2+ | LEDs + resistores | Indicação de estado |
| 1 | Botão / chave | Armar e desarmar o sistema |
| — | Protoboard, jumpers, fonte de alimentação | Montagem e energia |

## Esquema de ligação

> Os pinos abaixo são um exemplo. Atualize conforme o seu circuito e o seu código.

| Dispositivo | Pino do Arduino |
|-------------|-----------------|
| Receptor da barreira laser (LDR) | `A0` |
| Sensor PIR | `D2` |
| Botão armar/desarmar | `D3` |
| Buzzer | `D8` |
| Atuador (relé) | `D9` |
| LED de estado (armado) | `D10` |
| LED de alerta (disparo) | `D11` |

Adicione aqui a imagem do esquema elétrico:

```
docs/esquema_eletrico.png
```

## Lógica de funcionamento

O firmware é organizado como uma **máquina de estados**:

| Estado | Descrição |
|--------|-----------|
| `DESARMADO` | Sensores ignorados. O sistema aguarda o comando de armamento. |
| `ARMADO` | Monitoramento ativo. Leitura contínua da barreira laser e do PIR. |
| `DISPARADO` | Violação detectada. Alertas sonoros e atuadores acionados. |

**Fluxo resumido:**

1. O sistema inicia em `DESARMADO`.
2. Ao acionar o botão, passa para `ARMADO` e começa a monitorar os sensores.
3. Se o feixe do laser for interrompido **ou** o PIR detectar movimento, ocorre o evento de violação.
4. O sistema entra em `DISPARADO`: buzzer e atuadores são acionados de forma sincronizada.
5. O alarme só retorna ao estado `DESARMADO` mediante ação do usuário (botão/chave).

**Exemplo ilustrativo da estrutura do código:**

```cpp
enum Estado { DESARMADO, ARMADO, DISPARADO };
Estado estadoAtual = DESARMADO;

void loop() {
  verificarBotao();

  switch (estadoAtual) {
    case DESARMADO:
      desligarAlertas();
      break;

    case ARMADO:
      if (feixeInterrompido() || movimentoDetectado()) {
        estadoAtual = DISPARADO;
      }
      break;

    case DISPARADO:
      acionarAlertas();   // buzzer + atuadores sincronizados
      break;
  }
}
```

## Estrutura do repositório

> Adapte à organização real do seu projeto.

```
.
├── src/
│   └── alarme_perimetro.ino   # Firmware principal
├── docs/
│   ├── esquema_eletrico.png   # Diagrama de ligação
│   └── relatorio.pdf          # Documentação do trabalho
├── img/                       # Fotos e prints do protótipo
└── README.md
```

## Como executar

**Pré-requisitos**

- [Arduino IDE](https://www.arduino.cc/en/software) (ou PlatformIO)
- Cabo USB para o Arduino

**Passos**

1. Clone o repositório:
   ```bash
   git clone https://github.com/seu-usuario/seu-repositorio.git
   ```
2. Monte o circuito conforme a seção [Esquema de ligação](#esquema-de-ligação).
3. Abra o arquivo `src/alarme_perimetro.ino` na Arduino IDE.
4. Selecione a placa e a porta em **Ferramentas**.
5. Clique em **Carregar** (Upload).
6. Alinhe o laser com o receptor, arme o sistema e teste.

## Testes e calibração

- **Alinhamento do laser:** posicione o feixe diretamente sobre o LDR. Evite luz ambiente intensa incidindo no sensor.
- **Limiar do LDR:** ajuste o valor de referência no código após medir a leitura com o feixe incidindo e com o feixe interrompido.
- **Tempo de estabilização do PIR:** aguarde cerca de 30 a 60 segundos após ligar o sistema antes de testar.
- **Cenários de teste sugeridos:**
  - Interrupção do feixe com o sistema armado
  - Movimento na frente do PIR com o sistema armado
  - Verificação de que nada dispara no estado desarmado
  - Rearmamento após um disparo

## Limitações e melhorias futuras

- Sensibilidade do LDR à iluminação ambiente
- Alcance e alinhamento limitados do laser
- Possíveis evoluções:
  - Notificações remotas (Wi-Fi/GSM, ex.: ESP32 ou módulo SIM)
  - Teclado ou senha para armar e desarmar
  - Registro de eventos (cartão SD ou memória)
  - Bateria de backup para falta de energia
  - Câmera para registro de imagem do evento

## Tecnologias

- Plataforma **Arduino**
- Linguagem **C/C++**
- Arduino IDE
- Eletrônica básica: sensores ópticos, sensores de presença e atuadores

## Autores

- **Nome do Autor** – [GitHub](https://github.com/seu-usuario)
- Instituição / Curso / Disciplina: _preencher_
- Orientador(a): _preencher_

## Licença

Este projeto é de caráter acadêmico. Defina aqui a licença desejada (por exemplo, [MIT](https://opensource.org/licenses/MIT)).
