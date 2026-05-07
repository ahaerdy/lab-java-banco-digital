# 🏦 Banco Digital — Projeto Java POO

> Projeto desenvolvido como parte do curso **TQI - FullStack Developer** na plataforma [DIO](https://www.dio.me/), com o objetivo de praticar os quatro pilares da **Programação Orientada a Objetos (POO)** em Java: **abstração**, **encapsulamento**, **herança** e **polimorfismo**.

---

## Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Arquitetura e Design](#-arquitetura-e-design)
- [Descrição Detalhada dos Arquivos](#-descrição-detalhada-dos-arquivos)
  - [IConta.java — A Interface (Contrato)](#icontajava--a-interface-contrato)
  - [Conta.java — A Classe Abstrata (Base)](#contajava--a-classe-abstrata-base)
  - [ContaCorrente.java — Conta Corrente](#contaCorrentejava--conta-corrente)
  - [ContaPoupanca.java — Conta Poupança](#contapoupancajava--conta-poupança)
  - [Cliente.java — O Cliente](#clientejava--o-cliente)
  - [Banco.java — O Banco](#bancojava--o-banco)
  - [Main.java — Ponto de Entrada](#mainjava--ponto-de-entrada)
- [Os Quatro Pilares da POO no Projeto](#-os-quatro-pilares-da-poo-no-projeto)
- [Modificadores de Acesso](#-modificadores-de-acesso)
- [Saída Esperada](#-saída-esperada)
- [Como Executar](#-como-executar)
- [O que NÃO Versionar](#-o-que-não-versionar)
- [Tecnologias Utilizadas](#-tecnologias-utilizadas)
- [Referências](#-referências)

---

## Sobre o Projeto

Este projeto simula um sistema bancário digital simplificado. O domínio bancário foi escolhido pela sua familiaridade no dia a dia, o que facilita a tradução de regras de negócio para código.

**Funcionalidades implementadas:**

- Criação de clientes
- Abertura de Conta Corrente e Conta Poupança
- Depósito, saque e transferência entre contas
- Impressão de extrato detalhado por tipo de conta

---

## 📁 Estrutura do Projeto

```
lab-java-banco-digital/
├── src/
│   └── main/
│       └── java/
│           └── org/
│               └── banco/
│                   ├── IConta.java          ← Interface (contrato das contas)
│                   ├── Conta.java           ← Classe abstrata (base comum)
│                   ├── ContaCorrente.java   ← Tipo concreto de conta
│                   ├── ContaPoupanca.java   ← Tipo concreto de conta
│                   ├── Cliente.java         ← Entidade cliente
│                   ├── Banco.java           ← Entidade banco
│                   └── Main.java            ← Ponto de entrada da aplicação
├── .gitignore
└── README.md
```

> O caminho `src/main/java/org/banco` segue a convenção do Maven (gerenciador de projetos Java mais popular). A pasta `src/main/java` separa o código-fonte de testes e recursos. O pacote `org.banco` evita conflito de nomes com classes de outros projetos — é uma boa prática usar o domínio invertido da organização como prefixo do pacote.

---

## Arquitetura e Design

O diagrama abaixo ilustra o relacionamento entre as classes:

```
          «interface»
           IConta
        ┌─────────────────────┐
        │ + sacar()           │
        │ + depositar()       │
        │ + transferir()      │
        │ + imprimirExtrato() │
        └──────┬──────────────┘
               │ implements
        ┌──────┴───────┐
        │  «abstract»  │
        │    Conta     │         Cliente
        │──────────────│        ┌────────┐
        │ # agencia    │◄───────│ - nome │
        │ # numero     │ has-a  └────────┘
        │ # saldo      │
        │ # cliente    │
        └──────┬───────┘
               │ extends
      ┌────────┴─────────┐
      │                  │
┌─────┴───────┐   ┌──────┴──────┐
│ContaCorrente│   │ContaPoupanca│
└─────────────┘   └─────────────┘
```

> **Leitura do diagrama:** `IConta` define o **contrato** (o que toda conta deve saber fazer). `Conta` **implementa** esse contrato com a lógica padrão, mas delega `imprimirExtrato()` às subclasses. `ContaCorrente` e `ContaPoupanca` **herdam** de `Conta` e cada uma implementa o seu próprio extrato. `Cliente` é **composto** dentro de `Conta` (relação "tem-um").

---

## Descrição Detalhada dos Arquivos

---

### `IConta.java` — A Interface (Contrato)

```java
package org.banco;

public interface IConta {

    void sacar(double valor);
    void depositar(double valor);
    void transferir(double valor, IConta contaDestino);
    void imprimirExtrato();
}
```

**O que é uma interface?**
Uma interface é um **contrato**. Ela declara **o que** uma classe deve fazer, sem dizer **como** fazer. Qualquer classe que assine esse contrato (usando `implements`) é obrigada a implementar todos os métodos declarados.

**Por que usar uma interface aqui?**
Ao receber um `IConta` como parâmetro (em vez de `ContaCorrente` ou `ContaPoupanca` diretamente), o método `transferir` pode trabalhar com **qualquer tipo de conta** — presente ou futura. Se amanhã surgir uma `ContaInvestimento`, ela também pode ser usada em transferências sem alterar nenhum código existente. Isso é o **Princípio Aberto/Fechado** (Open/Closed Principle) do SOLID.

> Pense na interface como a tomada elétrica de uma casa. Qualquer aparelho com o plugue correto (que "implementa" o contrato) pode ser ligado, independente de ser uma geladeira, televisão ou ventilador.

---

### `Conta.java` — A Classe Abstrata (Base)

```java
package org.banco;

public abstract class Conta implements IConta {

    // Constante privada: só esta classe enxerga o valor da agência padrão.
    // "static final" indica que é compartilhada por TODAS as instâncias (nível de classe)
    // e nunca pode ser alterada após a inicialização.
    private static final int AGENCIA_PADRAO = 1;

    // Contador estático: compartilhado entre TODAS as contas.
    // Cada nova conta incrementa este número, garantindo que o número
    // de cada conta seja único e sequencial.
    private static int SEQUENCIAL = 1;

    // Atributos "protected": visíveis apenas para esta classe e suas subclasses.
    // Isso protege o saldo de ser alterado diretamente de fora da hierarquia.
    protected int agencia;
    protected int numero;
    protected double saldo;
    protected Cliente cliente;

    // Construtor: executado automaticamente ao criar qualquer conta.
    // Atribui a agência padrão e um número sequencial único, depois
    // incrementa o SEQUENCIAL para a próxima conta.
    public Conta(Cliente cliente) {
        this.agencia = Conta.AGENCIA_PADRAO;
        this.numero = SEQUENCIAL++;   // Atribui e depois incrementa (pós-incremento)
        this.cliente = cliente;
    }

    // Implementação padrão do saque: subtrai o valor do saldo.
    // A anotação @Override confirma que este método implementa
    // o que foi declarado na interface IConta.
    @Override
    public void sacar(double valor) {
        saldo -= valor;   // Equivalente a: saldo = saldo - valor
    }

    @Override
    public void depositar(double valor) {
        saldo += valor;   // Equivalente a: saldo = saldo + valor
    }

    // A transferência reutiliza os métodos já implementados:
    // 1) Saca da conta de origem (this = esta própria conta)
    // 2) Deposita na conta de destino
    // Repare que "contaDestino" é do tipo IConta — não importa se é
    // Corrente ou Poupança, desde que implemente a interface.
    @Override
    public void transferir(double valor, IConta contaDestino) {
        this.sacar(valor);
        contaDestino.depositar(valor);
    }

    // Getters públicos: permitem leitura controlada dos atributos protegidos.
    public int getAgencia() {
        return agencia;
    }

    public int getNumero() {
        return numero;
    }

    public double getSaldo() {
        return saldo;
    }

    // Método utilitário protegido: as subclasses chamam este método
    // para exibir os dados comuns, evitando duplicação de código (DRY).
    // String.format funciona como um template: %s = texto, %d = inteiro, %.2f = decimal com 2 casas.
    protected void imprimirInfosComuns() {
        System.out.println(String.format("Titular: %s", this.cliente.getNome()));
        System.out.println(String.format("Agencia: %d", this.agencia));
        System.out.println(String.format("Numero: %d", this.numero));
        System.out.println(String.format("Saldo: %.2f", this.saldo));
    }
}
```

**Por que a classe é `abstract`?**
Porque não faz sentido instanciar uma "Conta genérica" — no mundo real, toda conta é **Corrente** ou **Poupança**. A palavra-chave `abstract` impede que alguém escreva `new Conta(cliente)`, forçando o uso sempre de um tipo concreto. Além disso, o método `imprimirExtrato()` não tem implementação aqui porque cada tipo de conta exibe seu extrato de forma diferente — essa decisão é delegada às subclasses.

**O que é `static` em `SEQUENCIAL`?**
Atributos `static` pertencem à **classe**, não à instância. Isso significa que todas as contas compartilham o mesmo `SEQUENCIAL`. Quando a primeira conta é criada, `SEQUENCIAL` vai para 2; na segunda, para 3; e assim por diante. Se `SEQUENCIAL` não fosse `static`, cada conta teria seu próprio contador, e todas teriam número 1.

---

### `ContaCorrente.java` — Conta Corrente

```java
package org.banco;

public class ContaCorrente extends Conta {

    // O construtor recebe um cliente e simplesmente repassa ao
    // construtor da classe pai (Conta) usando "super()".
    // Toda a lógica de agência e número sequencial fica centralizada em Conta.
    public ContaCorrente(Cliente cliente) {
        super(cliente);
    }

    // Implementação concreta do método abstrato herdado.
    // Imprime o cabeçalho específico de Conta Corrente e, em seguida,
    // chama o método da classe pai para exibir os dados comuns.
    @Override
    public void imprimirExtrato() {
        System.out.println("=== Extrato Conta Corrente ===");
        super.imprimirInfosComuns();
    }
}
```

**O que é `extends`?**
`extends` estabelece uma relação de **herança**: `ContaCorrente` é uma especialização de `Conta`. Ela herda todos os atributos e métodos da classe pai e pode adicionar ou sobrescrever comportamentos. O comando `super(cliente)` chama o construtor da classe pai — em Java, isso deve ser a **primeira instrução** do construtor filho.

**O que é `@Override`?**
É uma anotação que instrui o compilador a verificar se o método está de fato sobrescrevendo algo da superclasse ou interface. Se o nome estiver errado (ex.: `imprimirextrato` com letra minúscula), o compilador avisa em vez de criar silenciosamente um método novo sem nenhum efeito.

---

### `ContaPoupanca.java` — Conta Poupança

```java
package org.banco;

public class ContaPoupanca extends Conta {

    public ContaPoupanca(Cliente cliente) {
        super(cliente);
    }

    // Mesmo padrão de ContaCorrente, mas com cabeçalho diferente.
    // Cada subclasse "sabe" como se apresentar sem alterar a lógica central.
    @Override
    public void imprimirExtrato() {
        System.out.println("=== Extrato Conta Poupança ===");
        super.imprimirInfosComuns();
    }
}
```

> **Nota de design:** A diferença entre `ContaCorrente` e `ContaPoupanca` neste projeto está apenas no extrato. Em um sistema real, a Conta Poupança teria regras extras, como rendimento mensal baseado na taxa SELIC e restrições no número de saques mensais.

---

### `Cliente.java` — O Cliente

```java
package org.banco;

public class Cliente {

    // Atributo privado: ninguém fora desta classe acessa "nome" diretamente.
    // Isso é encapsulamento — o estado interno fica protegido.
    private String nome;

    // Getter: método público para LER o nome.
    public String getNome() {
        return nome;
    }

    // Setter: método público para ESCREVER o nome.
    // No futuro, validações (ex.: verificar se o nome não é nulo ou vazio)
    // podem ser adicionadas aqui sem impactar quem chama o método.
    public void setNome(String nome) {
        this.nome = nome;
    }
}
```

**Por que não deixar o atributo `nome` público?**
Se `nome` fosse `public`, qualquer parte do código poderia escrever `cliente.nome = ""` ou `cliente.nome = null`, corrompendo o estado do objeto. Com o setter, é possível adicionar validações centralizadas:

```java
// Exemplo de como o setter poderia evoluir com validação:
public void setNome(String nome) {
    if (nome == null || nome.isBlank()) {
        throw new IllegalArgumentException("Nome do cliente não pode ser vazio.");
    }
    this.nome = nome;
}
```

Isso é a essência do **encapsulamento**: expor apenas o necessário e proteger a consistência interna.

---

### `Banco.java` — O Banco

```java
package org.banco;

import java.util.List;

public class Banco {

    private String nome;

    // List<Conta> é uma coleção genérica de contas.
    // O uso de "Conta" (tipo abstrato) em vez de "ContaCorrente" permite
    // armazenar qualquer tipo de conta na mesma lista.
    private List<Conta> contas;

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public List<Conta> getContas() {
        return contas;
    }

    public void setContas(List<Conta> contas) {
        this.contas = contas;
    }
}
```

**Estado atual e potencial de evolução:**
Nesta versão, `Banco` funciona como um repositório de dados. Em uma evolução natural do projeto, métodos de negócio poderiam ser adicionados aqui, por exemplo:

```java
// Métodos que poderiam ser adicionados futuramente:

public void adicionarConta(Conta conta) {
    this.contas.add(conta);
}

public Conta buscarPorNumero(int numero) {
    return this.contas.stream()
        .filter(c -> c.getNumero() == numero)
        .findFirst()
        .orElse(null);
}
```

---

### `Main.java` — Ponto de Entrada

```java
package org.banco;

public class Main {

    public static void main(String[] args) {

        // 1. Cria o cliente e define seu nome via setter.
        Cliente venilton = new Cliente();
        venilton.setNome("Venilton");

        // 2. Instancia os dois tipos de conta passando o cliente.
        //    Note que a variável é declarada como "Conta" (tipo abstrato),
        //    mas o objeto criado é do tipo concreto (ContaCorrente/ContaPoupanca).
        //    Isso é polimorfismo: cc pode ser tratado como qualquer Conta.
        Conta cc       = new ContaCorrente(venilton);
        Conta poupanca = new ContaPoupanca(venilton);

        // 3. Deposita R$ 100,00 na conta corrente.
        //    Saldo da CC passa de 0,00 para 100,00.
        cc.depositar(100);

        // 4. Transfere os R$ 100,00 da CC para a Poupança.
        //    Internamente: cc.sacar(100) e poupanca.depositar(100).
        //    Saldo da CC volta a 0,00. Saldo da Poupança vai para 100,00.
        cc.transferir(100, poupanca);

        // 5. Imprime os extratos.
        //    Cada objeto executa SEU PRÓPRIO imprimirExtrato() — polimorfismo em ação.
        cc.imprimirExtrato();
        poupanca.imprimirExtrato();
    }
}
```

**Fluxo de execução passo a passo:**

```
Criar cliente "Venilton"
        │
        ▼
Criar ContaCorrente (número: 1, saldo: 0,00)
        │
        ▼
Criar ContaPoupanca (número: 2, saldo: 0,00)
        │
        ▼
cc.depositar(100) → saldo CC = 100,00
        │
        ▼
cc.transferir(100, poupanca)
   ├── cc.sacar(100)         → saldo CC = 0,00
   └── poupanca.depositar(100) → saldo Poupança = 100,00
        │
        ▼
cc.imprimirExtrato()      → exibe extrato da Conta Corrente
poupanca.imprimirExtrato() → exibe extrato da Conta Poupança
```

---

## Os Quatro Pilares da POO no Projeto

| Pilar | Definição | Como aparece neste projeto |
|---|---|---|
| **Abstração** | Representar apenas o essencial de um conceito, ignorando detalhes irrelevantes | `IConta` define o contrato mínimo de qualquer conta. `Conta` implementa o comportamento comum sem se preocupar com o tipo específico |
| **Encapsulamento** | Proteger o estado interno de um objeto, expondo apenas o necessário | `saldo` é `protected`; só é alterado pelos métodos `sacar`, `depositar` e `transferir`. `SEQUENCIAL` é `private`, impedindo manipulação externa do contador |
| **Herança** | Reaproveitar comportamento de uma classe em outra mais especializada | `ContaCorrente` e `ContaPoupanca` herdam de `Conta` todos os atributos e métodos — sem duplicar uma linha de código |
| **Polimorfismo** | Tratar objetos de tipos diferentes de forma uniforme, confiando que cada um executará o comportamento correto | `cc` é declarada como `Conta`, mas em tempo de execução é uma `ContaCorrente`. `cc.imprimirExtrato()` invoca automaticamente a implementação correta. O mesmo vale para o parâmetro `IConta contaDestino` em `transferir()` |

---

## Modificadores de Acesso

Os modificadores controlam **quem pode ver e usar** cada membro de uma classe. A escolha correta é fundamental para o encapsulamento.

| Modificador | Visibilidade | Uso neste projeto |
|---|---|---|
| `private` | Somente a própria classe | `AGENCIA_PADRAO`, `SEQUENCIAL`, `nome` (Cliente), `nome` e `contas` (Banco) |
| `protected` | A classe, subclasses e classes do mesmo pacote | `agencia`, `numero`, `saldo`, `cliente` e `imprimirInfosComuns()` em `Conta` |
| `public` | Qualquer classe do sistema | Getters, setters, construtores e os métodos da interface |

**Regra prática:** comece sempre com `private`. Promova para `protected` apenas se uma subclasse precisar de acesso direto. Promova para `public` apenas para a API que o mundo externo deve enxergar. Nunca exponha atributos diretamente como `public`.

---

## Saída Esperada

Ao executar `Main.java`, a saída no console será:

```
=== Extrato Conta Corrente ===
Titular: Venilton
Agencia: 1
Numero: 1
Saldo: 0,00

=== Extrato Conta Poupança ===
Titular: Venilton
Agencia: 1
Numero: 2
Saldo: 100,00
```

> **Por que o saldo da Conta Corrente é `0,00`?** Porque o depósito de R$ 100,00 foi **integralmente transferido** para a Conta Poupança logo em seguida. O extrato reflete o saldo no momento da impressão.

---

## Como Executar

**Pré-requisito:** JDK 11 ou superior instalado.

```bash
# 1. Clone o repositório
git clone https://github.com/seu-usuario/lab-java-banco-digital.git
cd lab-java-banco-digital

# 2. Compile todos os arquivos Java (de dentro da pasta src/main/java)
javac -d out src/main/java/org/banco/*.java

# 3. Execute a classe principal
java -cp out org.banco.Main
```

Ou simplesmente abra o projeto no **IntelliJ IDEA** e pressione `Shift + F10` para compilar e executar.

---

## O que NÃO Versionar

O IntelliJ IDEA gera arquivos locais que não devem ir para o repositório. O `.gitignore` deve conter no mínimo:

```
out/
*.iml
.idea/
```

| Entrada | Motivo para ignorar |
|---|---|
| `out/` | Arquivos `.class` compilados. São gerados automaticamente e variam por máquina |
| `*.iml` | Configuração de módulo do IntelliJ, específica da instalação local |
| `.idea/` | Pasta de configurações internas da IDE, também local |

> **Regra de ouro do `.gitignore`:** versione apenas o que outro desenvolvedor precisaria para recriar o projeto do zero — o código-fonte e arquivos de configuração do projeto. Nunca versione artefatos gerados automaticamente.

---

## 🛠️ Tecnologias Utilizadas

- **Java** (JDK 11+)
- **IntelliJ IDEA Community Edition**

---

## 📚 Referências

- [Documentação oficial do Java — Oracle](https://docs.oracle.com/en/java/)
- [Repositório de Estudos — Bootcamp TQI Fullstack Developer](https://github.com/ahaerdy/DIO-learning/tree/main/TQI%20Fullstack%20Developer)