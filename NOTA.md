Esse exemplo mostra uma combinação de:

* **coroutines** (`co_yield`)
* **generator**
* **ranges/views**
* lazy evaluation em C++ moderno

O código:

```cpp
#include <experimental/generator>
#include <ranges>

std::experimental::generator<char> letters(char first)
{
    for (;; co_yield first++);
}

void generator_experiment()
{
    for (const char ch : letters('a') | std::views::take(26))
        std::cout << ch << ' ';

    std::cout << '\n';
}
```

faz isto:

```text
a b c d e f g ... z
```

---

# 1. O que é generator?

A ideia é parecida com Python:

Python:

```python
def letters():
    c = 'a'
    while True:
        yield c
        c += 1
```

C++:

```cpp
std::experimental::generator<char>
```

significa:

```text
"função que produz valores aos poucos"
```

---

# 2. Esta linha

```cpp
std::experimental::generator<char> letters(char first)
```

retorna um objeto que:

```text
gera chars sob demanda
```

---

# 3. O papel do `co_yield`

Aqui:

```cpp
for (;; co_yield first++);
```

temos:

```cpp
for (;;)
    co_yield first++;
```

ou seja:

```text
loop infinito
produz:
a
b
c
d
...
```

---

# 4. Como funciona o `co_yield`

Quando executa:

```cpp
co_yield first++;
```

o generator:

```text
entrega um valor
pausa execução
guarda estado interno
```

Depois, quando alguém pede o próximo valor:

```text
retoma exatamente dali
```

---

# 5. O loop é infinito

Isto:

```cpp
for (;;)
```

é:

```cpp
while(true)
```

Então o generator nunca termina sozinho.

---

# 6. Então por que não trava?

Porque você usa:

```cpp
std::views::take(26)
```

---

# 7. O pipeline ranges

Esta parte:

```cpp
letters('a') | std::views::take(26)
```

é um pipeline lazy.

Equivale a:

```text
gere letras infinitamente
↓
pegue apenas 26
```

---

# 8. O for final

```cpp
for (const char ch : ...)
```

vai receber:

```text
a b c ... z
```

---

# 9. Fluxo completo

```text
letters('a')
↓
generator infinito:
a b c d e ...
↓
views::take(26)
↓
a..z
↓
for imprime
```

---

# 10. A parte MAIS importante

O generator:

```text
não cria container
não aloca vetor
não gera tudo antecipadamente
```

Ele produz:

```text
1 valor por vez
```

Isso é:

```text
lazy evaluation
```

---

# 11. Mentalidade moderna

Isso se aproxima muito de:

* Haskell
* Rust iterators
* Python generators
* streams funcionais

---

# 12. Equivalente manual sem generator

Sem coroutine você faria algo parecido com:

```cpp
char c = 'a';

for(int i=0; i<26; ++i)
{
    std::cout << c++ << ' ';
}
```

Mas o generator permite:

```text
composição
pipelines
ranges
streams infinitos
```

---

# 13. Isso é poderoso para:

* parsing
* streams
* leitura de arquivos
* eventos
* sockets
* pipelines
* DSLs
* jogos
* simulação
* IA
* processamento incremental

---

# 14. Observação importante

```cpp
<experimental/generator>
```

ainda é experimental.

Hoje muitos compiladores usam:

```cpp
std::generator
```

(C++23/C++26 dependendo da implementação).

---

# 15. Visualização mental

Pense assim:

```text
letters('a')

┌────────────┐
│ estado='a' │
└─────┬──────┘
      │
 co_yield 'a'
      │ pausa
      ▼
 próximo pedido
      │
 retoma
      │
 co_yield 'b'
      │ pausa
      ▼
 ...
```

É literalmente uma máquina de estado automática gerada pelo compilador.

