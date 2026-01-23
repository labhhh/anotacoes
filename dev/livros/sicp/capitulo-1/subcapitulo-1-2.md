---
updated-at: 22/01/2026 21:42
created-at: 15/12/2025 10:35
---

> [!example] Tabela de conteúdos
> - [1.2. Funções e os processos que elas geram](#1.2.%20Funções%20e%20os%20processos%20que%20elas%20geram)
> 	- [1.2.1. Recursão linear e iteração](#1.2.1.%20Recursão%20linear%20e%20iteração)
> 	- [1.2.2. Recursão em árvore](#1.2.2.%20Recursão%20em%20árvore)

# 1.2. Funções e os processos que elas geram

- Funções são padrões para a evolução local de um processo computacional, especificando como cada parte do processo é realizada a partir da anterior;
- Essa seção exemplificará alguns dos padrões mais comuns em processos gerados por funções;

## 1.2.1. Recursão linear e iteração

- Considere a função fatorial:

$$n!=n\cdot(n-1)\cdot(n-2)~...~3\cdot2\cdot1$$

> [!info]
> Função fatorial é uma função matemática que consiste na multiplicação de um número natural por todos inteiros positivos menores do que ele.

- Podemos computar essa função matemática observando que, para qualquer número inteiro e positivo $n$, $n!$ é igual a $n$ vezes $(n-1)!$:

$$n!=n\cdot[(n-1)\cdot(n-2)~...~3\cdot2\cdot1]=n\cdot(n-1)!$$

- Assim, computar $n!$ é o mesmo que computar $(n-1)!$ e multiplicá-lo por $n$;
- Podemos traduzir essa afirmação para uma função Scheme:

```scheme
$> (define (factorial n)
	 (if (= n 1)
		 1
		 (* n (factorial (- n 1)))))
$> (factorial 4)
;? (* 4 (factorial 3))
;? (* 4 (* 3 (factorial 2)))
;? (* 4 (* 3 (* 2 (factorial 1))))
;? (* 4 (* 3 (* 2 1)))
;? (* 4 (* 3 2))
;? (* 4 6)
; 24
```

- Usando o modelo de substituição de avaliação de ordem normal podemos visualizar o processo de se computar $4!$.

---

- Essa não é a única forma de se traduzir $n!$ para Scheme, podemos multiplicar $n$ por um contador que vai de 1 à $n$, e finalizarmos quando o contador for maior que $n$:

```scheme
$> (define (factorial n)
	 (define (iter product counter)
	   (if (> counter n)
		   product
		   (iter (* counter product)
				 (+ counter 1))))
	 (iter 1 1))
$> (factorial 4)
;? (iter 1 1)
;? (iter 1 2)
;? (iter 2 3)
;? (iter 6 4)
;? (iter 24 5)
; 24
```

- Nessa abordagem, o contador e o produto do cálculo são atualizados simultaneamente entre os passos.

---

- Ambas funções computam o valor de $n!$, obtendo o mesmo número de produtos parciais (i.e. operações "incompletas");
- Porém, note como a execução da primeira implementação de `factorial` é expandida e reduzida;
- Essa expansão resulta em uma cadeia de **operações adiadas** (_deferred operations_) que, nesse caso, é a cadeia de multiplicações de $n!$;
	- Um processo formado por essa cadeia de operações é chamado de **processo recursivo** (_recursive process_).
- Executar esse tipo de processo requer que o interpretador gerencie quais operações serão executadas posteriormente;
- Nesse contexto, ao computarmos $n!$, a quantidade de multiplicações adiadas e a quantidade de informação necessária para a gerencia do processo são proporcionais ao valor de $n$.
	- O nome desse tipo de processo é **processo linearmente recursivo** (_linear recursive process_).

---

- Em contrapartida, a execução da segunda implementação de `factorial` não expande e reduz, isso porque, para cada $n$ passo, gerenciamos o valor das variáveis `product` e `counter`;
- Esse tipo de processo é chamado de **processo iterativo** (_iterative process_), nele, o estado da execução é representado pelos valores fixos de **variáveis de estado** (_state variables_) e de regras que descrevem como essas variáveis devem ser atualizadas a cada passo (i.e. estado);
	- Tais regras podem ter um caso-base que finaliza o processo.
- Nesse contexto, ao computarmos $n!$, a quantidade de multiplicações é proporcional ao valor de $n$.
	- O nome desse tipo de processo é **processo linearmente iterativo** (_linear iterative process_).

---

- No caso iterativo, as variáveis da função fornecem uma descrição completa do estado do processo em qualquer ponto da execução;
	- Caso interrompessemos a execução, poderíamos recuperar seu estado apenas passando valores para as variáveis.
- Já no caso recursivo, essas informações são gerenciadas pelo interpretador e o estado é a cadeia de operações adiadas.

---

- Funções recursivas e processos recursivos são diferentes, enquanto uma é a capacidade de uma função definir a si mesma, a outra é a forma (formato) que um processo cresce, respectivamente;
- Essa diferenciação pode ser confusa, isso porque a maioria das implementações de linguagens comuns (e.g., C, Java e Python) são projetadas de uma forma que qualquer função recursiva consome uma quantidade de memória que cresce junto com o número de execuções, mesmo que o processo seja iterativo;
	- Para criar iterações nessas linguagens são usadas instruções como `for`, `do`, `while`, etc.
- Nas implementações Scheme, mesmo que uma função seja recursiva, ela será executada como um processo iterativo em um espaço constante[^1];
	- Esse tipo de implementação é chamado de **função de fim recursivo** (_tail-recursive_), nela, processos iterativos podem ser realizados mesmo por meio de funções recursivas.

> [!info]
> - Em funções de fim recursivo, a última operação feita deve ser a chamada recursiva da função, sem nenhuma computação pendente a ser realizada;
> - A função `iter` do caso iterativo de `factorial` é _tail-recursive_, isso porque, se o contador não for maior que `n`, `iter` executa a si mesmo;
> - Na avaliação de `(iter (* counter product) (+ counter 1))`, as subexpressões `(* counter product)` e `(+ counter 1)` são avaliadas primeiro, e então `iter` é executada;
> - No caso recursivo de `factorial`, em `(* n (factorial (- n 1)))`, a execução de `factorial` não acontece até que o caso-base seja alcançado.

## 1.2.2. Recursão em árvore

- Considere a sequência:

$$0,1,1,2,3,5,8,13,21~...$$

- Essa é a sequência de Fibonacci, onde cada número é a soma dos dois anteriores, começando em 0 e 1;
- Os números da sequência de Fibonacci são definidos pela regra:

$$
\begin{equation}
	Fib(n) = \begin{cases}
		   ~0 & \text{se} ~ n = 0, \\
		   ~1 & \text{se} ~ n = 1, \\
		   ~Fib(n-1) + Fib(n-2) & \text{caso contrário}.
	\end{cases}
\end{equation}
$$

- Podemos traduzir essa análise de caso para Scheme:

```scheme
$> (define (fib n)
     (cond ((= n 0) 0)
           ((= n 1) 1)
           (else (+ (fib (- n 1))
                    (fib (- n 2))))))
$> (fib 5)
;? (+ (fib (- 5 1)) (fib (- 5 2)))
;? (+ (fib 4) (fib 3))
;? (+ (+ (fib (- 4 1)) (fib (- 4 2))) (+ (fib (- 3 1)) (fib (- 3 2))))
;? (+ (+ (fib 3) (fib 2)) (+ (fib 2) (fib 1)))
;? (+ (+ (+ (fib (- 3 1)) (fib (- 3 2))) (+ (fib (- 2 1)) (fib (- 2 2)))) (+ (+ (fib (- 2 1)) (fib (- 2 2))) 1))
;? (+ (+ (+ (fib 2) (fib 1)) (+ (fib 1) (fib 0))) (+ (+ (fib 1) (fib 0)) 1))
;? (+ (+ (+ (fib 2) 1) (+ 1 0)) (+ (+ 1 0) 1))
;? (+ (+ (+ (+ (fib (- 2 1)) (fib (- 2 2))) 1) (+ 1 0)) (+ (+ 1 0) 1))
;? (+ (+ (+ (+ (fib 1) (fib 0)) 1) (+ 1 0)) (+ (+ 1 0) 1))
;? (+ (+ (+ (+ 1 0) 1) 1) (+ 1 1))
;? (+ (+ (+ 1 1) 1) 2)
;? (+ (+ 2 1) 2)
;? (+ 3 2)
; 5
```

- Na função `fib`, para computar `(fib 5)`, computamos `(fib 4)` e `(fib 3)`;
- Para computar `(fib 4)`, computamos `(fib 3)` e `(fib 2)`, e assim até chegarmos nos casos-base;
- Note que o processo evolui como uma árvore, onde cada execução é dividida "galhos":

```
                                               (fib 5)
                               ___________________|___________________
                              /                                       \
                          (fib 4)                                   (fib 3)
                  ___________|__________                         ______|______
                 /                      \                       /             \
             (fib 3)                  (fib 2)               (fib 2)         (fib 1)
           _____|______               ___|___               ___|___            |       
          /            \             /       \             /       \          (1)
      (fib 2)        (fib 1)     (fib 1)   (fib 0)     (fib 1)   (fib 0)
     ___|___            |           |         |           |         |
    /       \          (1)         (1)       (0)         (1)       (0)
(fib 1)   (fib 0)
   |         |
  (1)       (0)
```

- Essa divisão acontece porque a função `fib` chama a si mesmo duas vezes a cada execução;
- Porém, essa implementação de `fib` foi feita apenas para ilustrar uma recursão em árvore, havendo muita computação redundante;
	- Por exemplo, `(fib 3)`, que representa metade da computação, é executado 2 vezes.
- Os caso-base de `fib` são as operações `(fib 1)` e/ou `(fib 0)`, assim, o número de execuções delas é $Fib(n+1)$;
- $Fib(n)$ cresce exponencialmente com o valor de $n$, sendo o número inteiro mais próximo de $\phi^n/\sqrt{5}$;
- $\phi$ é o valor da **proporção áurea** (_golden ratio_):

$$
\phi=\frac{1+\sqrt{5}}{2}\approx1.6180
$$

- why $\phi^2=\phi+1$?
- $1.6180\cdot1.6180=2.6179$
- $1.6180+1=2.618$

# parei em
https://web.mit.edu/6.001/6.037/sicp.pdf#page=48
https://sicp.sourceacademy.org/chapters/1.2.2.html#p3

[^1]: CLINGER, William. [Proper Tail Recursion and Space Efficiency](https://www.cs.tufts.edu/~nr/cs257/archive/will-clinger/proper-tail-space.pdf). Acesso em 16/12/2025.
