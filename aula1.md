# Aula 1 — Design Patterns: O Vocabulário Oculto e o Padrão Strategy

**Pré-requisitos:** Aulas de OOP (classes, métodos, composição), Flask (rotas, `@app.route`), JavaScript (funções, eventos), Testes (pytest).

**Objetivo da aula:** entender o que são Design Patterns, reconhecer os padrões que você já usa no dia a dia (mesmo sem saber o nome deles) e aplicar o padrão **Strategy** para eliminar blocos gigantes de `if/elif` em regras de negócio.

> **Definição rápida:** **Design Pattern (Padrão de Projeto)** é uma solução geral e reutilizável para um problema recorrente no design de software. Não é código pronto para copiar e colar; é um **modelo** ou **vocabulário** compartilhado entre desenvolvedores.

---

## 0. Introdução: O que é (e o que não é) um Design Pattern

### 0.1 A analogia da construção

Quando um engenheiro civil projeta uma casa, ele não precisa reinventar o conceito de "porta", "janela" ou "alicerce" a cada novo projeto. Ele usa soluções que já foram testadas, documentadas e nomeadas pela indústria.

No software, acontece a mesma coisa. Desde os anos 90, desenvolvedores começaram a catalogar soluções para problemas que apareciam repetidamente em projetos diferentes. O livro mais famoso sobre o assunto, escrito por quatro autores (conhecidos como "Gang of Four" ou GoF), catalogou 23 padrões clássicos.

### 0.2 O que Design Patterns NÃO são

- ❌ **Não são bibliotecas ou frameworks:** você não instala um padrão via `pip`.
- ❌ **Não são regras obrigatórias:** aplicar um padrão onde ele não é necessário gera *over-engineering* (código complexo demais para um problema simples).
- ❌ **Não são código pronto:** eles descrevem a *estrutura* da solução, não a implementação exata.

### 0.3 Por que aprender isso agora?

Porque os padrões funcionam como **vocabulário técnico**. Em vez de dizer:

> *"Eu criei uma função que recebe outra função como parâmetro e executa ela dentro de um loop para filtrar a lista..."*

Você pode dizer:

> *"Eu apliquei o padrão Strategy para isolar as regras de filtro."*

Isso acelera a comunicação em equipes e facilita a leitura de documentação técnica.

---

## 1. Os Padrões que Você Já Usa (A Grande Revelação)

Antes de aprender padrões novos, vamos nomear os que você já aplicou ao longo do curso. Python, JavaScript e Flask já embutem vários padrões clássicos em sua sintaxe, poupando-nos do *boilerplate* (código repetitivo) exigido por linguagens como Java ou C++.

### 1.1 MTV / MVC (Arquitetura de Camadas)

**Onde vimos:** Em todas as aulas de Flask.

O padrão **MVC (Model-View-Controller)** separa a aplicação em três responsabilidades:
- **Model:** dados e regras de negócio (ex: classe `Tarefa`, `TarefaRepository`).
- **View:** apresentação visual (ex: templates HTML `base.html`).
- **Controller:** lógica de roteamento e coordenação (ex: funções decoradas com `@app.route`).

O Django e o Flask usam uma variação chamada **MTV (Model-Template-View)**, mas a ideia é a mesma: separação de responsabilidades.

| Camada | Responsabilidade | Exemplo no curso |
|---|---|---|
| Model | Dados, validação, persistência | `class Tarefa`, `TarefaRepository` |
| Template | Apresentação HTML | `templates/index.html` |
| View/Controller | Roteamento HTTP, orquestração | `@app.route('/api/tarefas')` |

**Por que importa:** O HTML não sabe SQL; o SQL não sabe HTTP. Essa separação permite que você teste cada camada isoladamente (como fizemos na Aula 2 de Testes).

**Referência:**
- [Flask Documentation - Tutorial](https://flask.palletsprojects.com/en/stable/tutorial/) (mostra a estrutura de pastas `templates/`, `static/` e a separação de responsabilidades)

---

### 1.2 Factory (Fábrica)

**Onde vimos:** Na Aula 2 de Testes, com a função `criar_app(config_extra)`.

O padrão **Factory** (Fábrica) encapsula a lógica de criação de objetos complexos. Em vez de espalhar `Flask(__name__)` por todo o código, você chama uma função que "fabrica" e devolve uma instância configurada.

**Antes (sem Factory):**
```python
# app.py
app = Flask(__name__)
app.config["DATABASE"] = "tarefas.db"

@app.route("/")
def index():
    return "Olá"

if __name__ == "__main__":
    app.run()
```

**Depois (com Factory):**
```python
# app.py
def criar_app(config_extra=None):
    app = Flask(__name__)
    app.config["DATABASE"] = "tarefas.db"
    
    if config_extra:
        app.config.update(config_extra)
    
    @app.route("/")
    def index():
        return "Olá"
    
    return app

if __name__ == "__main__":
    app = criar_app()
    app.run()
```


<details>
<summary>Não exagere:</summary>

Um conto interessante. Tradução livre de [Factory Factory Factory](https://notes.zachmanson.com/factory-factory-factory/)

---

Vamos fingir que decidi construir uma estante porta-temperos.

Já fiz pequenos projetos de marcenaria antes e acho que tenho uma boa ideia do que vou precisar: um pouco de madeira e algumas ferramentas básicas — uma trena, um serrote, um nível e um martelo.

Se eu fosse construir uma casa inteira, e não apenas um porta-temperos, continuaria precisando de uma trena, um serrote, um nível e um martelo (entre outras coisas).

Então vou à loja de material de construção comprar as ferramentas e pergunto ao vendedor onde posso encontrar um martelo.

"Um martelo?", ele pergunta. "Ninguém mais compra martelo hoje em dia. É meio antiquado."

Surpreso com essa novidade, pergunto o motivo.

"Bom, o problema dos martelos é que existem tantos tipos diferentes. Marreta, martelo-unha, martelo de pena. E se você comprar um tipo e depois perceber que precisa de outro? Vai ter que comprar um martelo separado pra cada tarefa nova. Acontece que, no fim das contas, a maioria das pessoas quer mesmo é um único martelo que dê conta de todo tipo de martelada que a vida pode exigir."

"Hmmmmm. Bom, acho que faz sentido. Você pode me mostrar onde encontro um Martelo Universal?"

"Não, a gente não vende mais esses. Já estão bem obsoletos."

"Sério? Pensei que você tinha acabado de dizer que o Martelo Universal era a onda do futuro."

"Acontece que, se você fabrica um único tipo de martelo, capaz de executar todas as mesmas tarefas que todos aqueles tipos diferentes, ele acaba não sendo muito bom em nenhuma delas. Pregar um prego com uma marreta não é lá muito eficiente. E, se você quiser matar alguém, não existe substituto pra um martelo de pena."

"Faz sentido. Então, se ninguém mais compra Martelo Universal e vocês não vendem mais aqueles tipos antiquados, que tipo de martelo vocês vendem?"

"Na verdade, a gente não vende martelo nenhum."

"Então..."

"Segundo nossa pesquisa, o que as pessoas realmente precisavam não era de um Martelo Universal, afinal. É sempre melhor ter o tipo certo de martelo pra cada trabalho. Então começamos a vender fábricas de martelos, capazes de produzir qualquer tipo de martelo que você queira usar. Tudo o que você precisa fazer é contratar funcionários pra fábrica, ligar as máquinas, comprar a matéria-prima, pagar as contas de luz e água, e PRESTO... você vai ter exatamente o martelo de que precisa num piscar de olhos."

"Mas eu não quero comprar uma fábrica de martelos..."

"Ainda bem. Porque a gente não vende mais elas também."

"Mas eu achei que você tinha acabado de..."

"Descobrimos que a maioria das pessoas, na verdade, não precisa de uma fábrica de martelos inteira. Algumas pessoas, por exemplo, nunca vão precisar de um martelo de pena. (Talvez nunca tenham precisado matar alguém. Ou talvez tenham matado com picadores de gelo.) Então não faz sentido alguém comprar uma fábrica que produza todo tipo de martelo que existe debaixo do sol."

"É, faz bastante sentido."

"Então, em vez disso, começamos a vender plantas e diagramas de fábricas de martelos, permitindo que nossos clientes construíssem suas próprias fábricas, projetadas sob medida pra fabricar apenas os tipos de martelo de que realmente vão precisar."

"Deixa eu adivinhar. Vocês não vendem mais isso também."

"Exato. Não vendemos, não. Acontece que as pessoas não querem construir uma fábrica inteira só pra fabricar dois ou três martelos. Deixa a construção da fábrica com os especialistas em construir fábricas, é o que eu sempre digo!!"

"E eu concordo com você nesse ponto."

"Pois é. Então paramos de vender as plantas e começamos a vender fábricas de fábricas de martelos. Cada fábrica de fábricas de martelos é construída pra você pelos maiores especialistas no ramo de fábricas de fábricas de martelos, então você não precisa se preocupar com todos os detalhes envolvidos na construção de uma fábrica. Mesmo assim, você ainda aproveita todos os benefícios de ter sua própria fábrica de martelos personalizada, despejando seus próprios martelos personalizados, segundo suas próprias especificações."

"Bom, isso na verdade não..."

"Eu sei o que você vai dizer!! ...e a gente não vende mais esses também. Por algum motivo, quase ninguém estava comprando as fábricas de fábricas de martelos, então bolamos uma solução nova pra resolver o problema."

"Aham."

"Quando demos um passo pra trás e analisamos a infraestrutura global de ferramentas, concluímos que as pessoas estavam frustradas por ter que gerenciar e operar uma fábrica de fábricas de martelos, além da fábrica de martelos que ela produzia. Esse tipo de sobrecarga fica bem pesado quando você lida com o cenário provável de também operar uma fábrica de fábricas de trenas, uma fábrica de fábricas de serrotes e uma fábrica de fábricas de níveis, sem falar num conglomerado de empresa holding de fabricação de madeira. Quando realmente analisamos a situação, concluímos que é complexo demais pra alguém que só quer construir um porta-temperos."

"É, sem brincadeira."

"Então, esta semana, estamos lançando uma fábrica de fábricas de fábricas de ferramentas de uso geral, pra que todas as suas diferentes fábricas de fábricas de ferramentas possam ser produzidas por uma única fábrica unificada. A fábrica de fábricas de fábricas vai produzir apenas as fábricas de fábricas de ferramentas de que você realmente precisa, e cada uma dessas fábricas de fábricas vai produzir uma única fábrica com base nas suas especificações personalizadas. O conjunto final de ferramentas que emerge desse processo será o conjunto ideal pro seu projeto específico. Você vai ter exatamente o martelo de que precisa e exatamente a trena certa pra sua tarefa, tudo com o apertar de um botão (embora talvez você também precise implantar alguns arquivos de configuração pra que tudo funcione conforme suas expectativas)."

"Então vocês não têm nenhum martelo? Nenhum mesmo?"

"Não. Se você realmente quer um porta-temperos de alta qualidade, projetado industrialmente, precisa desesperadamente de algo mais avançado do que um simples martelo de uma lojinha mixuruca de bairro."

"E é assim que todo mundo faz agora? Todo mundo usa uma fábrica de fábricas de fábricas de ferramentas de uso geral sempre que precisa de um martelo?"

"Sim."

"Bom, tudo bem. Acho que é isso que vou ter que fazer. Se é assim que as coisas funcionam hoje em dia, é melhor eu aprender a fazer assim."

"Muito bem!!"

"Isso vem com manual, né?"

---

</details>

**Python vs GoF:** O livro original exige classes abstratas, interfaces formais e métodos estáticos complexos (`AbstractFactory`, `ConcreteFactory`). Em Python, **uma simples função que retorna um objeto configurado já é o padrão Factory**. Não precisamos de hierarquia de classes para isso.

**Referência:**
- [Flask Documentation - Application Factories](https://flask.palletsprojects.com/en/stable/patterns/appfactories/)

---

### 1.3 Decorator (Decorador)

**Onde vimos:** `@app.route('/api/tarefas')`, `@pytest.fixture`, `@login_required`.

O padrão **Decorator** permite adicionar comportamento a um objeto ou função dinamicamente, sem alterar sua estrutura original.

**O que o `@` faz por baixo dos panos:**

```python
# O que você escreve:
@app.route("/api/tarefas")
def listar_tarefas():
    return []

# O que o Python faz internamente (açúcar sintático):
def listar_tarefas():
    return []

listar_tarefas = app.route("/api/tarefas")(listar_tarefas)
```

O método `app.route()` recebe a função `listar_tarefas`, registra ela no mapa de rotas do Flask e devolve a função original (ou uma versão "decorada" com metadados).

**Python vs GoF:** O padrão GoF original usa composição de classes e delegação. Python simplificou isso com a sintaxe `@` e o conceito de *Higher-Order Functions* (funções que recebem ou retornam outras funções).

**Referência:**
- [Python Documentation - PEP 318: Decorators for Functions and Methods](https://peps.python.org/pep-0318/)

---

### 1.4 Singleton (Instância Única)

**Onde vimos:** Nos `imports` do Python e no `current_app` do Flask.

O padrão **Singleton** garante que uma classe tenha apenas uma instância e fornece um ponto de acesso global a ela.

**Java/C++ (forma clássica):**
```java
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
    private DatabaseConnection() { } // Construtor privado
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}
```

**Python (forma natural):**
Em Python, **todo módulo importado é um Singleton natural**. O Python carrega o módulo na memória uma única vez e reutiliza a mesma referência em todos os `imports`.

```python
# config.py
DATABASE_URL = "sqlite:///tarefas.db"
```

```python
# modulo_a.py
import config
print(config.DATABASE_URL)
```

```python
# modulo_b.py
import config
print(config.DATABASE_URL)  # Mesma referência em memória
```

**`current_app` do Flask:**
O `current_app` é um *proxy* (um objeto que delega chamadas) para a aplicação Flask que está processando a requisição atual. Ele garante que, mesmo em ambientes com múltiplas threads, cada requisição acesse a instância correta da aplicação.

**Por que não usamos Singleton manual em Python:**
A forma clássica do GoF esconde o construtor (`__init__`) e força o uso de métodos estáticos (`get_instance()`). Isso dificulta testes (você não consegue injetar uma instância falsa) e quebra o princípio de injeção de dependências. Módulos globais já resolvem o problema de forma mais simples e testável.

**Referência:**
- [Flask Documentation - Application Context](https://flask.palletsprojects.com/en/stable/appcontext/)

---

## 2. O Novo Problema: A Classe "Deus" e o `if/elif` Gigante

Agora que nomeamos o que você já sabe, vamos introduzir um padrão novo para resolver um problema real que você provavelmente já enfrentou.

### 2.1 O cenário

Imagine que você está construindo um sistema de e-commerce e precisa calcular o preço final de um produto aplicando descontos. A regra de negócio inicial é simples:

```python
# carrinho.py
class Carrinho:
    def __init__(self, subtotal):
        self.subtotal = subtotal

    def calcular_total(self):
        # Sem desconto por enquanto
        return self.subtotal
```

Aí o marketing pede: "Vamos fazer uma promoção de Black Friday com 20% de desconto". Você altera o código:

```python
class Carrinho:
    def __init__(self, subtotal, eh_black_friday=False):
        self.subtotal = subtotal
        self.eh_black_friday = eh_black_friday

    def calcular_total(self):
        if self.eh_black_friday:
            return self.subtotal * 0.80  # 20% de desconto
        return self.subtotal
```

Semana seguinte: "Agora temos cupons de 10% para clientes VIP". Você adiciona mais um parâmetro e mais um `if`:

```python
class Carrinho:
    def __init__(self, subtotal, eh_black_friday=False, eh_cliente_vip=False):
        self.subtotal = subtotal
        self.eh_black_friday = eh_black_friday
        self.eh_cliente_vip = eh_cliente_vip

    def calcular_total(self):
        if self.eh_black_friday:
            return self.subtotal * 0.80
        elif self.eh_cliente_vip:
            return self.subtotal * 0.90
        return self.subtotal
```

Mês seguinte: "Temos promoção de Natal com 15% de desconto, mas só para compras acima de R$ 200". O código vira isso:

```python
class Carrinho:
    def __init__(
        self,
        subtotal,
        eh_black_friday=False,
        eh_cliente_vip=False,
        eh_natal=False
    ):
        self.subtotal = subtotal
        self.eh_black_friday = eh_black_friday
        self.eh_cliente_vip = eh_cliente_vip
        self.eh_natal = eh_natal

    def calcular_total(self):
        if self.eh_black_friday:
            return self.subtotal * 0.80
        elif self.eh_cliente_vip:
            return self.subtotal * 0.90
        elif self.eh_natal and self.subtotal > 200:
            return self.subtotal * 0.85
        return self.subtotal
```

### 2.2 Os problemas desse código

1. **Violação do Princípio Aberto/Fechado (Open/Closed Principle):** Toda vez que surge uma nova promoção, você precisa **modificar** a classe `Carrinho`. Se você introduzir um bug no `elif` do Natal, pode quebrar a Black Friday e o Cliente VIP juntos.

2. **Acoplamento:** A classe `Carrinho` agora sabe detalhes de todas as promoções do sistema. Se o marketing decidir que a promoção de Natal deve ser 18% em vez de 15%, você precisa abrir o arquivo do carrinho e alterar a lógica.

3. **Dificuldade de testar:** Para testar a promoção de Natal, você precisa instanciar um carrinho com `eh_natal=True` e `subtotal=250`. O teste fica verboso e difícil de ler.

4. **Impossibilidade de combinar regras:** E se o cliente for VIP **e** for Black Friday? O código atual aplica apenas a primeira regra que encontrar (Black Friday), ignorando o VIP. Você precisaria de uma lógica complexa de priorização.

---

## 3. A Solução: O Padrão Strategy (Estratégia)

O padrão **Strategy** propõe isolar cada algoritmo (cada regra de desconto) em sua própria "caixa" e deixar o objeto principal (`Carrinho`) apenas "apontar" para a caixa certa em tempo de execução.

### 3.1 Conceito

> **Definição rápida:** **Strategy** define uma família de algoritmos, encapsula cada um deles e os torna intercambiáveis. O cliente (a classe `Carrinho`) pode escolher qual algoritmo usar sem conhecer os detalhes internos.

### 3.2 Aplicação em Python: Funções como Cidadãos de Primeira Classe

Em linguagens como Java, o Strategy é implementado com interfaces e classes concretas (`interface DescontoStrategy`, `class BlackFridayStrategy implements DescontoStrategy`). Isso gera muito *boilerplate*.

Em Python, **funções são objetos de primeira classe**. Isso significa que você pode passar uma função como argumento, retornar uma função de outra função e armazenar funções em variáveis. Vamos usar isso para implementar o Strategy de forma limpa.

**Refatorando o código:**

```python
# descontos.py

def desconto_black_friday(subtotal):
    """Aplica 20% de desconto."""
    return subtotal * 0.80


def desconto_cliente_vip(subtotal):
    """Aplica 10% de desconto."""
    return subtotal * 0.90


def desconto_natal(subtotal):
    """Aplica 15% de desconto apenas para compras acima de R$ 200."""
    if subtotal > 200:
        return subtotal * 0.85
    return subtotal


def sem_desconto(subtotal):
    """Nenhum desconto aplicado."""
    return subtotal
```

Agora, a classe `Carrinho` recebe a **estratégia** como parâmetro:

```python
# carrinho.py
from descontos import sem_desconto

class Carrinho:
    def __init__(self, subtotal, estrategia_desconto=sem_desconto):
        self.subtotal = subtotal
        self.estrategia_desconto = estrategia_desconto

    def calcular_total(self):
        return self.estrategia_desconto(self.subtotal)
```

**Como usar:**

```python
from carrinho import Carrinho
from descontos import desconto_black_friday, desconto_natal

# Sem desconto (padrão)
c1 = Carrinho(100.0)
print(c1.calcular_total())  # 100.0

# Com desconto de Black Friday
c2 = Carrinho(100.0, desconto_black_friday)
print(c2.calcular_total())  # 80.0

# Com desconto de Natal (mas subtotal abaixo de 200)
c3 = Carrinho(100.0, desconto_natal)
print(c3.calcular_total())  # 100.0 (não aplicou desconto)

# Com desconto de Natal (subtotal acima de 200)
c4 = Carrinho(300.0, desconto_natal)
print(c4.calcular_total())  # 255.0
```

Saída esperada:

```text
100.0
80.0
100.0
255.0
```

### 3.3 Comparação antes/depois

| Aspecto | Antes (if/elif gigante) | Depois (Strategy) |
|---|---|---|
| Adicionar nova promoção | Modificar a classe `Carrinho` | Criar nova função em `descontos.py` |
| Testar uma promoção | Instanciar carrinho com flag `eh_natal=True` | Chamar `desconto_natal(100)` diretamente |
| Combinar regras | Lógica complexa de priorização | Compor funções ou criar nova estratégia |
| Princípio Aberto/Fechado | Violado (mexer em código antigo) | Respeitado (apenas adicionar código novo) |

### 3.4 Aplicação em JavaScript: Passar Funções para Componentes

O mesmo padrão se aplica no frontend. Imagine um componente que renderiza uma lista de tarefas. Você quer poder filtrar essa lista de diferentes formas (apenas pendentes, apenas concluídas, todas).

**Sem Strategy:**

```javascript
// listaTarefas.js
function renderizarLista(tarefas, filtro) {
    const lista = document.getElementById('lista');
    lista.innerHTML = '';

    for (const tarefa of tarefas) {
        let deveMostrar = false;

        if (filtro === 'todas') {
            deveMostrar = true;
        } else if (filtro === 'pendentes') {
            deveMostrar = !tarefa.concluida;
        } else if (filtro === 'concluidas') {
            deveMostrar = tarefa.concluida;
        }

        if (deveMostrar) {
            const li = document.createElement('li');
            li.textContent = tarefa.descricao;
            lista.appendChild(li);
        }
    }
}
```

**Com Strategy:**

```javascript
// filtros.js
function mostrarTodas(tarefa) {
    return true;
}

function mostrarApenasPendentes(tarefa) {
    return !tarefa.concluida;
}

function mostrarApenasConcluidas(tarefa) {
    return tarefa.concluida;
}
```

```javascript
// listaTarefas.js
function renderizarLista(tarefas, estrategiaFiltro) {
    const lista = document.getElementById('lista');
    lista.innerHTML = '';

    for (const tarefa of tarefas) {
        if (estrategiaFiltro(tarefa)) {
            const li = document.createElement('li');
            li.textContent = tarefa.descricao;
            lista.appendChild(li);
        }
    }
}

// Uso:
const tarefas = [
    { descricao: 'Estudar', concluida: false },
    { descricao: 'Trabalhar', concluida: true },
];

renderizarLista(tarefas, mostrarApenasPendentes);
```

---

## 4. Exercícios

### Exercício 1 — Strategy para cálculo de frete

Crie um arquivo `frete.py` com três estratégias de cálculo de frete:
1. `frete_gratis(valor)` - retorna 0.0
2. `frete_fixo(valor)` - retorna 15.0 (independente do valor da compra)
3. `frete_proporcional(valor)` - retorna 10% do valor da compra

Crie uma classe `Pedido` em `pedido.py` que recebe `subtotal` e `estrategia_frete` (padrão: `frete_fixo`) e tem um método `calcular_total()` que retorna `subtotal + frete`.

<details>
<summary>Ver solução</summary>

```python
# frete.py
def frete_gratis(valor):
    return 0.0


def frete_fixo(valor):
    return 15.0


def frete_proporcional(valor):
    return valor * 0.10
```

```python
# pedido.py
from frete import frete_fixo

class Pedido:
    def __init__(self, subtotal, estrategia_frete=frete_fixo):
        self.subtotal = subtotal
        self.estrategia_frete = estrategia_frete

    def calcular_total(self):
        frete = self.estrategia_frete(self.subtotal)
        return self.subtotal + frete
```

Testando:

```python
from pedido import Pedido
from frete import frete_gratis, frete_proporcional

p1 = Pedido(100.0)  # frete fixo padrão
print(p1.calcular_total())  # 115.0

p2 = Pedido(100.0, frete_gratis)
print(p2.calcular_total())  # 100.0

p3 = Pedido(100.0, frete_proporcional)
print(p3.calcular_total())  # 110.0
```
</details>

---

### Exercício 2 — Testando estratégias isoladamente

Usando o código do Exercício 1, escreva testes unitários com `pytest` que:
1. Testem a função `frete_gratis` diretamente (sem precisar instanciar `Pedido`).
2. Testem a função `frete_proporcional` para um valor de 200.0 (deve retornar 20.0).
3. Testem o método `calcular_total` do `Pedido` com a estratégia `frete_gratis`.

<details>
<summary>Ver solução</summary>

```python
# test_frete.py
from frete import frete_gratis, frete_proporcional
from pedido import Pedido

def test_frete_gratis_retorna_zero():
    assert frete_gratis(100.0) == 0.0
    assert frete_gratis(0.0) == 0.0

def test_frete_proporcional_10_porcento():
    assert frete_proporcional(200.0) == 20.0
    assert frete_proporcional(50.0) == 5.0

def test_pedido_com_frete_gratis():
    p = Pedido(100.0, frete_gratis)
    assert p.calcular_total() == 100.0
```
</details>

---

### Exercício 3 — Strategy para validação de senha

Crie um arquivo `validadores.py` com três estratégias de validação de senha:
1. `validador_fraco(senha)` - aceita qualquer senha não vazia
2. `validador_medio(senha)` - exige mínimo de 8 caracteres
3. `validador_forte(senha)` - exige mínimo de 8 caracteres, pelo menos uma letra maiúscula e pelo menos um número

Crie uma classe `Usuario` em `usuario.py` que recebe `nome`, `senha` e `estrategia_validacao` (padrão: `validador_medio`) e tem um método `senha_eh_valida()` que retorna `True` ou `False`.

<details>
<summary>Ver solução</summary>

```python
# validadores.py
def validador_fraco(senha):
    return len(senha) > 0


def validador_medio(senha):
    return len(senha) >= 8


def validador_forte(senha):
    if len(senha) < 8:
        return False
    
    tem_maiuscula = any(char.isupper() for char in senha)
    tem_numero = any(char.isdigit() for char in senha)
    
    return tem_maiuscula and tem_numero
```

```python
# usuario.py
from validadores import validador_medio

class Usuario:
    def __init__(self, nome, senha, estrategia_validacao=validador_medio):
        self.nome = nome
        self.senha = senha
        self.estrategia_validacao = estrategia_validacao

    def senha_eh_valida(self):
        return self.estrategia_validacao(self.senha)
```

Testando:

```python
from usuario import Usuario
from validadores import validador_fraco, validador_forte

u1 = Usuario("João", "123", validador_fraco)
print(u1.senha_eh_valida())  # True

u2 = Usuario("Maria", "123")  # validador_medio por padrão
print(u2.senha_eh_valida())  # False

u3 = Usuario("Ana", "Senha123", validador_forte)
print(u3.senha_eh_valida())  # True
```
</details>

---

## 5. Erros Comuns

### Erro 1: Criar classes inteiras para estratégias simples

**Errado (over-engineering):**

```python
class EstrategiaDesconto:
    def calcular(self, subtotal):
        raise NotImplementedError


class BlackFridayEstrategia(EstrategiaDesconto):
    def calcular(self, subtotal):
        return subtotal * 0.80


class Carrinho:
    def __init__(self, subtotal, estrategia):
        self.subtotal = subtotal
        self.estrategia = estrategia

    def calcular_total(self):
        return self.estrategia.calcular(self.subtotal)
```

**Certo (Pythonico):**

```python
def desconto_black_friday(subtotal):
    return subtotal * 0.80


class Carrinho:
    def __init__(self, subtotal, estrategia_desconto=desconto_black_friday):
        self.subtotal = subtotal
        self.estrategia_desconto = estrategia_desconto

    def calcular_total(self):
        return self.estrategia_desconto(self.subtotal)
```

**Por que:** Em Python, funções são objetos e podem ser passadas diretamente. Criar uma classe inteira com um único método `calcular()` só adiciona *boilerplate* desnecessário. Use classes apenas se a estratégia precisar manter estado interno (o que é raro).

---

### Erro 2: Esquecer de passar a estratégia padrão

**Errado:**

```python
class Carrinho:
    def __init__(self, subtotal, estrategia_desconto):
        self.subtotal = subtotal
        self.estrategia_desconto = estrategia_desconto
```

Se alguém instanciar `Carrinho(100.0)` sem passar a estratégia, o código quebra com `TypeError`.

**Certo:**

```python
from descontos import sem_desconto

class Carrinho:
    def __init__(self, subtotal, estrategia_desconto=sem_desconto):
        self.subtotal = subtotal
        self.estrategia_desconto = estrategia_desconto
```

Sempre forneça um padrão sensato (como `sem_desconto` ou `validador_medio`) para que a classe funcione mesmo sem configuração explícita.

---

### Erro 3: Aplicar Strategy para uma única regra

**Errado:**

```python
# Só existe uma regra de desconto no sistema inteiro
def desconto_unico(subtotal):
    return subtotal * 0.90


class Carrinho:
    def __init__(self, subtotal):
        self.subtotal = subtotal

    def calcular_total(self):
        return desconto_unico(self.subtotal)
```

**Certo:**

```python
class Carrinho:
    def __init__(self, subtotal):
        self.subtotal = subtotal

    def calcular_total(self):
        # Regra fixa, sem variação
        return self.subtotal * 0.90
```

**Por que:** O padrão Strategy só faz sentido quando você tem **múltiplas variações** de um algoritmo e espera que novas variações surjam no futuro. Se existe apenas uma regra e ela nunca vai mudar, usar Strategy é *over-engineering*. Mantenha o código simples.

---

## 6. Quando usar / Quando não usar

### Use Strategy quando:

- ✅ Você tem **múltiplas variações** de um algoritmo (descontos, validações, filtros, ordenações).
- ✅ Você quer **adicionar novas variações sem modificar código existente** (Princípio Aberto/Fechado).
- ✅ Você quer **testar cada variação isoladamente** (sem precisar instanciar a classe principal).
- ✅ Você quer permitir que o **cliente escolha** qual algoritmo usar em tempo de execução.

### Não use Strategy quando:

- ❌ Existe apenas **uma variação** do algoritmo e ela nunca vai mudar.
- ❌ O algoritmo é **trivial** (uma linha de código).
- ❌ Você está aplicando o padrão "porque ouvi falar que é bom", não porque resolve um problema real.

---

## 7. Checklist da aula

Ao final desta aula, você deve conseguir:

- explicar o que é um Design Pattern e por que eles são úteis como vocabulário técnico;
- reconhecer os padrões **MVC/MTV**, **Factory**, **Decorator** e **Singleton** no código que você já escreveu ao longo do curso;
- identificar o problema do "if/elif gigante" em regras de negócio;
- aplicar o padrão **Strategy** usando funções como cidadãos de primeira classe em Python;
- aplicar o padrão **Strategy** em JavaScript passando funções como parâmetros;
- escrever testes unitários para estratégias isoladas (sem precisar instanciar a classe principal);
- reconhecer quando **não** aplicar Strategy (evitar over-engineering).

---

## 8. Recapitulação

| Conceito | Significado |
|---|---|
| **Design Pattern** | Solução geral reutilizável para um problema recorrente de design de software. |
| **MVC/MTV** | Separação da aplicação em Model (dados), View/Template (apresentação) e Controller (roteamento). |
| **Factory** | Função ou classe que encapsula a lógica de criação de objetos complexos. |
| **Decorator** | Padrão que adiciona comportamento a um objeto/função dinamicamente (sintaxe `@` em Python). |
| **Singleton** | Padrão que garante uma única instância de uma classe (módulos Python são Singletons naturais). |
| **Strategy** | Padrão que isola cada algoritmo em sua própria "caixa" e permite trocar entre eles em tempo de execução. |
| **Princípio Aberto/Fechado** | Classes devem estar abertas para extensão, mas fechadas para modificação. |
| **Over-engineering** | Aplicar padrões ou complexidade desnecessária para um problema simples. |

---

## 9. O que não vimos no curso

Os tópicos abaixo são comuns em catálogos de Design Patterns, mas ficaram de fora desta aula para manter o foco no que é pragmático para Python, JavaScript e Flask.

| Tópico | Síntese |
|---|---|
| **Strategy com classes (GoF clássico)** | O livro original usa interfaces e classes concretas (`interface Strategy`, `class ConcreteStrategy`). Em Python, funções puras são mais limpas e suficientes na maioria dos casos. |
| **Abstract Factory** | Padrão de criação que fornece uma interface para criar famílias de objetos relacionados. Exige hierarquia de classes complexa; em Python, funções factory simples resolvem 99% dos casos. |
| **Builder** | Padrão de criação que separa a construção de um objeto complexo da sua representação. Python permite construir objetos complexos de forma limpa usando `kwargs` e dicionários, tornando o Builder formal obsoleto na maioria dos casos. |
| **Prototype** | Padrão de criação que permite copiar objetos existentes. Python tem `copy.copy()` e `copy.deepcopy()` na biblioteca padrão, tornando o padrão formal desnecessário. |
| **State** | Padrão comportamental que permite que um objeto altere seu comportamento quando seu estado interno muda. Útil em máquinas de estado complexas, mas raramente necessário em aplicações web CRUD. |
| **Template Method** | Padrão comportamental que define o esqueleto de um algoritmo em uma classe base, deixando algumas etapas para subclasses. Em Python, composição e funções de *callback* são preferidas sobre herança profunda. |

---

## 10. Referências

### Livros

- **Design Patterns: Elements of Reusable Object-Oriented Software** — Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides (1994). O livro original dos 23 padrões do "Gang of Four". Denso e acadêmico, mas é a fonte primária.
- **Head First Design Patterns** — Eric Freeman, Elisabeth Robson (2020). Versão didática e visual dos padrões do GoF, com exemplos em Java. Excelente para iniciantes.

### Recursos Online

- [Refactoring.Guru - Design Patterns](https://refactoring.guru/design-patterns) — Catálogo visual e didático de todos os padrões, com exemplos em várias linguagens (incluindo Python). Melhor recurso online gratuito.
- [Refactoring.Guru - Strategy Pattern](https://refactoring.guru/design-patterns/strategy) — Página específica sobre o padrão Strategy com diagramas e exemplos.
- [Python Documentation - PEP 318: Decorators](https://peps.python.org/pep-0318/) — Proposta original que introduziu a sintaxe `@` no Python.
- [Flask Documentation - Application Factories](https://flask.palletsprojects.com/en/stable/patterns/appfactories/) — Documentação oficial sobre o padrão Factory no Flask.
- [Flask Documentation - Application Context](https://flask.palletsprojects.com/en/stable/appcontext/) — Explicação sobre `current_app` e como o Flask gerencia a instância da aplicação.

### Artigos

- [Martin Fowler - Inversion of Control](https://martinfowler.com/bliki/InversionOfControl.html) — Conceito relacionado que explica por que passar estratégias como parâmetros é melhor do que criar dependências rígidas.
- [Real Python - Python's Instance, Class, and Static Methods Demystified](https://realpython.com/instance-class-and-static-methods-demystified/) — Explica a diferença entre métodos de instância, de classe e estáticos, útil para entender por que funções puras são preferidas em muitos casos.