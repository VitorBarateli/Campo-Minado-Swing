# 💣 Campo Minado GUI - Java Swing & Event-Driven Architecture
Uma versão gráfica completa e moderna do clássico jogo **Campo Minado**, desenvolvida em Java utilizando a biblioteca nativa **Swing**. O projeto adota uma arquitetura orientada a eventos (*Event-Driven*) desacoplada, utilizando múltiplas camadas de observadores para garantir sincronia em tempo real entre a lógica de negócios e os componentes visuais.

---

## 📌 Sumário
- [Destaques e Recursos Visuais](#-destaques-e-recursos-visuais)
- [Arquitetura Orientada a Eventos](#-arquitetura-orientada-a-eventos)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Como Executar o Projeto](#-como-executar-o-projeto)
- [Regras e Mecânicas de Jogo](#-regras-e-mecânicas-de-jogo)

---

## 🚀 Destaques e Recursos Visuais

* **Interface Reativa Baseada em Grid:** O tabuleiro possui uma grade dinâmica de $16 \times 30$ (480 campos) com 50 minas sorteadas, gerenciado de forma fluida pelo `GridLayout`.
* **Estilização por Estados:** Cada botão (`BotaoCampo`) renderiza dinamicamente cores e fontes específicas baseadas no número de minas vizinhas (1 para Verde, 2 para Azul, 3 para Amarelo e valores superiores em Vermelho) ou se o campo foi marcado/explodido.
* **Segurança de Threads (SwingUtilities):** O processamento de eventos que alteram a UI é empacotado e despachado na fila de eventos apropriada do Swing através do `SwingUtilities.invokeLater()`, prevenindo *deadlocks* ou comportamentos assíncronos inesperados de renderização.
* **Propagação Recursiva Otimizada:** Varredura em lotes usando `parallelStream()` mapeia rapidamente cliques em áreas seguras, desencadeando a abertura visual em cascata instantaneamente.

---

## 🧠 Arquitetura Orientada a Eventos

A aplicação se destaca pela implementação inteligente do padrão **Observer**, subdividido em duas frentes de responsabilidade para manter o modelo isolado da interface:

```text
[ Modelo: Campo ] ────(CampoEvento)────> [ Visão: BotaoCampo ]
       │                                         │
 (Interação)                                 (Cliques)
       ▼                                         ▼
[ Modelo: Tabuleiro ] ─(ResultadoEvento)─> [ Visão: PainelTabuleiro ]
```
1. Observador Atômico (CampoObservador): Cada instância de Campo possui uma lista de observadores. O componente visual BotaoCampo se registra diretamente no seu respectivo campo. Quando o campo muda de estado (ABRIR, MARCAR, DESMARCAR, EXPLODIR), o botão é notificado e atualiza seu visual individual de forma cirúrgica, sem precisar redesenhar a tela inteira.
2. Observador Macro (ResultadoEvento): O Tabuleiro atua como agregador e implementa uma interface funcional baseada em Consumer<ResultadoEvento>. Quando a condição de vitória ou derrota é validada pelo motor do jogo, caixas de diálogo nativas (JOptionPane) são disparadas de forma assíncrona.

---

## 🗂️ Estrutura do Projeto
Os pacotes dividem estritamente as responsabilidades de visualização (GUI) e regras de domínio (Core):
```text
📂 src/
└── 📂 projeto/
    ├── 📂 modelo/
    │   ├── 📄 Campo.java              # Estado atômico da célula, vizinhos e gatilhos
    │   ├── 📄 CampoEvento.java        # Enumerador com os estados de alteração da célula
    │   ├── 📄 CampoObservador.java    # Interface funcional para os observadores individuais
    │   ├── 📄 Tabuleiro.java          # Concentrador da matriz, sorteios e regras de fim de jogo
    │   └── 📄 ResultadoEvento.java    # Transportador de dados de vitória/derrota
    └── 📂 visao/
        ├── 📄 TelaPrincipal.java      # Janela mestre da aplicação (JFrame)
        ├── 📄 PainelTabuleiro.java    # Painel container com o grid estruturado (JPanel)
        └── 📄 BotaoCampo.java         # Componente visual customizado reativo (JButton)
```

---

## 💻 Como Executar o Projeto
Pré-requisitos:
- Java Development Kit (JDK) 11 ou superior configurado no sistema.

Compilação e Execução via Terminal:
1. Clone o repositório:
```bash
git clone https://github.com/VitorBarateli/Campo-Minado-Swing.git
cd nome-do-repositorio
```
2. Compile os fontes:
Crie um diretório temporário de binários e compile todas as classes associadas:
```bash
javac -d bin src/projeto/modelo/*.java src/projeto/visao/*.java
```
3. Inicie a interface gráfica:
Execute a classe TelaPrincipal para instanciar o JFrame nativo do Campo Minado:
```bash
java -cp bin projeto.visao.TelaPrincipal
```

---

## 🎮 Regras e Mecânicas de Jogo
- Botão Esquerdo do Mouse: Abre o bloco selecionado. Se o bloco possuir uma bomba, ela explode e o jogo revela todos os outros blocos com bombas na tela imediatamente, finalizando a rodada.
- Botão Direito do Mouse: Alterna a marcação de segurança (Bandeira M). Use para indicar visualmente onde você suspeita fortemente que haja uma mina oculta.
- Condição de Vitória: O jogo analisa os predicados continuamente. A vitória é declarada quando todos os blocos seguros (sem minas) forem abertos e todas as posições perigosas estiverem devidamente sinalizadas.
