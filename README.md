# 🏦 Banco Digital — Projeto Java POO

Projeto desenvolvido como parte do curso **TQI - FullStack Developer** na plataforma [DIO](https://www.dio.me/), com o objetivo de praticar os conceitos de **Programação Orientada a Objetos (POO)** em Java, incluindo herança, abstração, encapsulamento e polimorfismo.

> **Instrutor:** Venilton Falvo Jr. — Doutor em Engenharia de Software e Education Tech Lead na DIO. [LinkedIn](https://www.linkedin.com/in/falvojr/)

---

## 📌 Descrição do Projeto

Simulação de um sistema bancário digital simples, onde é possível criar clientes, abrir contas correntes e contas poupança, realizar depósitos, saques e transferências entre contas, além de imprimir extratos detalhados.

O domínio bancário foi escolhido por sua familiaridade, facilitando a tradução de regras de negócio para o código. O desafio consiste em:

- **Entidades:** O banco oferece dois tipos de conta — **Corrente** e **Poupança**.
- **Funcionalidades:** As contas devem permitir operações de **depósito**, **saque** e **transferência** (restrita a contas da própria instituição).

---

## 🗂️ Estrutura dos Arquivos

### `IConta.java`
Interface que define o **contrato** de comportamento de qualquer conta bancária no sistema. Declara os seguintes métodos que todas as contas devem implementar:

- `sacar(double valor)` — Realiza um saque na conta.
- `depositar(double valor)` — Realiza um depósito na conta.
- `transferir(double valor, IConta contaDestino)` — Transfere um valor para outra conta.
- `imprimirExtrato()` — Exibe o extrato da conta.

---

### `Conta.java`
Classe **abstrata** que implementa a interface `IConta` e serve como base para os tipos concretos de conta. Contém:

- Atributos comuns a todas as contas: `agencia`, `numero`, `saldo` e `cliente`.
- Lógica de numeração automática de contas via atributo estático `SEQUENCIAL`.
- Agência padrão definida pela constante `AGENCIA_PADRAO = 1`.
- Implementação padrão dos métodos `sacar`, `depositar` e `transferir`.
- Método protegido `imprimirInfosComuns()`, utilizado pelas subclasses para exibir titular, agência, número e saldo.
- O método `imprimirExtrato()` é abstrato, delegando a implementação às subclasses.

---

### `ContaCorrente.java`
Subclasse concreta de `Conta` que representa uma **Conta Corrente**. Implementa o método `imprimirExtrato()`, exibindo o cabeçalho `=== Extrato Conta Corrente ===` seguido das informações comuns da conta.

---

### `ContaPoupanca.java`
Subclasse concreta de `Conta` que representa uma **Conta Poupança**. Implementa o método `imprimirExtrato()`, exibindo o cabeçalho `=== Extrato Conta Poupança ===` seguido das informações comuns da conta.

---

### `Cliente.java`
Classe que representa o **cliente** do banco. Possui apenas o atributo `nome`, com os métodos `getNome()` e `setNome()` para acesso e modificação do valor.

---

### `Banco.java`
Classe que representa a entidade **Banco**. Possui os atributos:

- `nome` — Nome do banco.
- `contas` — Lista de contas associadas ao banco (do tipo `List<Conta>`).

Fornece getters e setters para ambos os atributos. Nesta versão do projeto, a classe serve como estrutura de dados do banco, sem lógica de negócio adicional implementada.

---

### `Main.java`
Classe principal com o método `main`, ponto de entrada do programa. Demonstra o funcionamento do sistema:

1. Cria um cliente chamado **Arthur**.
2. Abre uma Conta Corrente e uma Conta Poupança para esse cliente.
3. Deposita R$ 100,00 na Conta Corrente.
4. Transfere R$ 100,00 da Conta Corrente para a Conta Poupança.
5. Imprime o extrato de ambas as contas.

---

## ▶️ Saída Esperada da Execução

```
=== Extrato Conta Corrente ===
Titular: Arthur
Agencia: 1
Numero: 1
Saldo: 0,00
=== Extrato Conta Poupança ===
Titular: Arthur
Agencia: 1
Numero: 2
Saldo: 100,00
```

> **Observação:** O saldo da Conta Corrente aparece como `0,00` porque o depósito de R$ 100,00 foi integralmente transferido para a Conta Poupança.

---

## 🧠 Conceitos de POO Aplicados

| Conceito | Aplicação no Projeto |
|---|---|
| **Abstração** | Classe abstrata `Conta` e interface `IConta` definem apenas o essencial do domínio bancário |
| **Encapsulamento** | Atributos `private`/`protected` protegem a lógica interna; o saldo só é alterado via `sacar`, `depositar` e `transferir` |
| **Herança** | `ContaCorrente` e `ContaPoupanca` herdam de `Conta`, aplicando o princípio DRY (Don't Repeat Yourself) |
| **Polimorfismo** | Objetos instanciados como `ContaCorrente` ou `ContaPoupanca` são referenciados pelo tipo `Conta`, e cada um executa seu próprio `imprimirExtrato()` |

### Sobre os modificadores de acesso utilizados

- **`private`** — Informação restrita à própria classe (ex.: constante `AGENCIA_PADRAO` e contador `SEQUENCIAL`).
- **`protected`** — Compartilhado dentro da hierarquia de herança; permite que as subclasses acessem `agencia`, `numero` e `saldo` sem expô-los publicamente.
- **`public`** — Acessível a qualquer classe do sistema (ex.: `getAgencia()`, `getSaldo()`).

---

## 📋 Requisitos Técnicos

- Conhecimentos básicos de Programação Orientada a Objetos em Java
- Ambiente de desenvolvimento Java configurado (JDK 11+)
- Familiaridade com repositórios Git (opcional, mas recomendado)
- Capacidade de abstração para reproduzir a solução proposta e implementar melhorias

---

## 🛠️ Tecnologias Utilizadas

- **Java** (JDK 11+)
- **IntelliJ IDEA Community Edition**

---

## 🚀 Sugestões de Evolução

O instrutor propõe as seguintes melhorias como desafio adicional:

- **Extensão do Domínio** — Incluir novas classes, atributos e métodos ao projeto (ex.: tipos de conta adicionais, taxas, rendimentos).
- **Redução de Verbosidade** — Incluir o [Lombok](https://projectlombok.org/) para eliminar código repetitivo como getters, setters e construtores.

---

## 🔗 Links Úteis

- [Slides da aula](https://docs.google.com/presentation/d/1sGnTlpJK0F08hSZebk8LNTsOkHVBivVu/edit?usp=sharing&ouid=105300330738120646134&rtpof=true&sd=true)
- [Repositório de referência do instrutor](https://github.com/falvojr/dio-live-20210802)
- [Curso na plataforma DIO](https://web.dio.me/project/criando-um-banco-digital-com-java-e-orientacao-objetos/learning/133f3c7e-4474-4582-8e78-987f038ab283)

---

## 👤 Autor

Desenvolvido por **Arthur** como entrega para o curso **TQI - FullStack Developer** na plataforma DIO.
