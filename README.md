# Documentação da ponderada CPU

## Visão Geral

Este projeto implementa uma **CPU sequencial simples** construída em lógica digital, com execução de instruções baseada em **memórias EEPROM**, controle por **clock manual** e armazenamento de resultados em registradores.

A CPU foi projetada para executar uma sequência fixa de operações aritméticas, utilizando:

- um **contador**
- uma **EEPROM de OPCODE**
- uma **EEPROM de operandos**
- uma **ULA (ALU)**
- dois registradores principais:
  - **AC (Acumulador)**
  - **MQ (registrador auxiliar / quociente / extensão de resultado)**

O objetivo principal é demonstrar como uma CPU pode ser montada a partir de componentes básicos de arquitetura de computadores e lógica digital.

---

# Objetivo da CPU

A CPU foi programada para executar, em ordem sequencial, a seguinte rotina matemática:

1. **Somar 1**
2. **Multiplicar por 2**
3. **Somar 3**
4. **Subtrair 1**
5. **Dividir por 3**

Ou seja, partindo de um valor inicial, a CPU deve executar essas instruções **uma por vez**, sincronizadas por clock.

---

# Funcionamento Geral

A execução acontece de forma **sequencial** e **sincronizada por clock**.

Cada vez que o botão de clock é pressionado:

1. o **contador** avança para o próximo endereço
2. esse endereço é enviado simultaneamente para:
   - a **EEPROM de OPCODE**
   - a **EEPROM de Operandos**
3. a **EEPROM de OPCODE** informa **qual operação** a ULA deve realizar
4. a **EEPROM de Operandos** informa **qual valor numérico** será usado naquela operação
5. a ULA executa a operação
6. o resultado é armazenado nos registradores **AC** e/ou **MQ**
7. esses registradores alimentam a próxima operação

Assim, a CPU executa uma **instrução por pulso de clock**, em ordem fixa.

---

# Estrutura da CPU

## 1. Clock manual

A CPU é acionada por um **botão de clock manual**.

Cada clique no botão representa **um ciclo de clock**, ou seja, **uma etapa de execução**.

Esse clock alimenta o contador e sincroniza a leitura das memórias e a atualização dos registradores.

---

## 2. Contador

O contador é responsável por **iterar os endereços das memórias**.

### Configuração:
- **4 bits**

### Função:
A cada pulso de clock, ele avança para o próximo endereço:

- `0000`
- `0001`
- `0010`
- `0011`
- ...

Como ele possui **4 bits**, ele pode acessar:

- **16 posições de memória** (`0x0` até `0xF`)

Esse mesmo contador é usado para acessar **simultaneamente**:

- a memória de instruções (**EEPROM OPCODE**)
- a memória de dados (**EEPROM de Operandos**)

---

# Memórias Utilizadas

A CPU utiliza **duas EEPROMs principais**:

---

## 3. EEPROM de OPCODE

Essa memória armazena **as instruções da CPU**, ou seja, informa **qual operação** deve ser realizada em cada ciclo.

### Configuração:
- **4 bits de endereço**
- **5 bits de dados**

### Função:
Cada endereço da EEPROM corresponde a uma instrução da CPU.

Exemplo:
- endereço `0x0` → operação de soma
- endereço `0x1` → operação de multiplicação
- etc.

Os **5 bits de saída** da EEPROM são divididos em sinais de controle para a ULA.

---

## 4. EEPROM de Operandos

Essa memória armazena os **valores numéricos** usados pelas operações.

### Configuração:
- **4 bits de endereço**
- **8 bits de dados**

### Função:
Cada endereço da EEPROM de operandos contém o valor que será usado junto com a instrução correspondente da EEPROM OPCODE.

Exemplo:
- se no endereço `0x1` a EEPROM OPCODE indica “multiplicação”
- então no endereço `0x1` da EEPROM de operandos estará o valor pelo qual multiplicar

---

# Execução Sequencial das Instruções

As instruções da CPU acontecem **de forma sequencial e sincronizada**.

Isso significa que:

> **a instrução no endereço X da EEPROM OPCODE acontece junto com o operando no endereço X da EEPROM de operandos**

Ou seja:

- **Operação 1** usa **Operando 1**
- **Operação 2** usa **Operando 2**
- **Operação 3** usa **Operando 3**
- e assim por diante

## Exemplo:
Se o contador estiver no endereço `0x2`, então:

- a CPU lê o **opcode do endereço `0x2`**
- a CPU lê o **operando do endereço `0x2`**
- e executa exatamente essa combinação

Essa lógica é o que faz a CPU se comportar como um sistema de instruções sequenciais.

---

# Registradores Principais

A CPU utiliza dois registradores para armazenar os resultados das operações da ULA:

---

## 5. Registrador AC

O **AC (Acumulador)** armazena o **resultado principal** da maioria das operações.

Ele representa o valor acumulado ao longo da execução.

Exemplo:
- após uma soma, o resultado vai para o AC
- esse valor será usado como base para a próxima operação

---

## 6. Registrador MQ

O **MQ** é um registrador auxiliar utilizado para armazenar saídas complementares da ULA, como por exemplo:

- parte alta de multiplicações
- quociente
- extensão de resultado
- sinais auxiliares da operação

Dependendo da operação executada, o MQ pode receber valores diferentes da ULA.

---

# Reutilização de Resultados: entrada A da próxima operação

Após cada operação, os resultados armazenados em **AC** e **MQ** são reutilizados como entrada da próxima instrução.

Para isso, é utilizado um **Splitter**.

---

## 7. Splitter

O splitter é utilizado para **organizar e redirecionar os bits** dos registradores de saída da ULA de forma adequada.

### Função principal:
- pegar os valores armazenados em **AC** e **MQ**
- organizá-los
- e reaproveitá-los como entrada **A** da próxima operação da ULA

Ou seja:

> o resultado de uma operação é utilizado como base da operação seguinte

Isso faz com que a CPU tenha comportamento **encadeado**, em vez de executar operações isoladas.

---

# ULA (ALU)

A **ULA (Unidade Lógica e Aritmética)** é o bloco responsável por realizar as operações matemáticas.

Ela recebe:

- **Entrada A** → valor vindo do acumulador / resultado anterior
- **Entrada B** → valor vindo da EEPROM de operandos
- **Sinais de controle** → vindos da EEPROM OPCODE

Com isso, a ULA executa a operação indicada.

---

# Controle da ULA via OPCODE

A EEPROM OPCODE fornece **5 bits de controle** para a ULA.

Esses 5 bits são divididos por um **splitter** em dois grupos:

- **Sel AC** → 3 bits
- **Sel MQ** → 2 bits

Esses seletores controlam os multiplexadores da ULA, determinando:

- qual operação será enviada ao registrador **AC**
- qual saída auxiliar será enviada ao registrador **MQ**

---

# OPCODES Utilizados

Os seguintes opcodes foram utilizados no projeto:

| Operação       | Opcode (5 bits) |
|----------------|-----------------|
| Soma           | `00000`         |
| Subtração      | `00100`         |
| Multiplicação  | `01001`         |
| Divisão        | `01110`         |

Esses códigos são armazenados na **EEPROM de OPCODE** e interpretados pela ULA.

---

# Programa Gravado nas EEPROMs

## EEPROM OPCODE

Esta memória define a sequência de operações da CPU.

| Endereço | Instrução       | Opcode   |
|----------|-----------------|----------|
| `0x0`    | Soma            | `00000`  |
| `0x1`    | Multiplicação   | `01001`  |
| `0x2`    | Soma            | `00000`  |
| `0x3`    | Subtração       | `00100`  |
| `0x4`    | Divisão         | `01110`  |

---

## EEPROM de Operandos

Esta memória define os valores numéricos usados em cada instrução.

| Endereço | Operando | Binário      |
|----------|----------|--------------|
| `0x0`    | `1`      | `00000001`   |
| `0x1`    | `2`      | `00000010`   |
| `0x2`    | `3`      | `00000011`   |
| `0x3`    | `1`      | `00000001`   |
| `0x4`    | `3`      | `00000011`   |

---

# Conclusão 

Esta CPU representa uma implementação didática e funcional de uma arquitetura sequencial simples, mostrando como componentes fundamentais como: contador, EEPROM, registradores, ULA, clock e multiplexadores podem ser integrados para executar uma sequência de instruções reais em hardware lógico.

# Vídeo explicativo
[![Assista ao vídeo](https://img.youtube.com/vi/d9IqE9SVTxc/hqdefault.jpg)](https://www.youtube.com/watch?v=d9IqE9SVTxc)


# Documentação da ponderada - ALU 

## Descrição

Este projeto consiste no desenvolvimento de uma ALU (Arithmetic Logic Unit) implementada em lógica digital.
A ALU é capaz de realizar operações aritméticas fundamentais entre dois operandos binários.

## Funcionalidades desenvolvidas

- Soma (Addition)
- Subtração (Subtraction)
- Multiplicação (Multiplication)
- Divisão (Division)
- Shift para a esquerda (Left Shift)
- Shift para a direita (Right Shift)


## Estrutura 

Full Adders - base para soma e subtração
Half Adders - base para multiplicação
Subtratores (via complemento de 2)
Multiplicador - baseado em operações combinacionais
Divisor - implementado com subtrações sucessivas
Shift - deslocamento de bits

## link do vídeo explicativo
detalhe: No minuto 7:32 eu chamo a saída da divisão R de razão, mas se trata do resto, devido ao vídeo ter sido gravado sem pausas (e com algumas tentativas) acabei me confundindo na hora e nem percebi o equívoco. 

link do vídeo: https://www.youtube.com/watch?v=llkG70HEZeg

[![Assista ao vídeo](https://img.youtube.com/vi/llkG70HEZeg/maxresdefault.jpg)](https://www.youtube.com/watch?v=llkG70HEZeg)
