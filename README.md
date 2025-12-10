# Relatório Acadêmico: Análise Teórica e Implementação Prática do Algoritmo de Shor

**Autores:** Lucas de Luccas, André Hutzler e Diogo Burgierman

**Data:** 10 de Dezembro de 2025

---

## 1. Introdução

[cite_start]O algoritmo de Shor, proposto em 1994 por Peter Shor, tornou-se um marco fundamental da computação quântica ao demonstrar que a fatoração de inteiros — considerada inviável em tempo hábil por métodos clássicos — pode ser resolvida em tempo polinomial em um computador quântico[cite: 5].

[cite_start]Essa descoberta ameaça diretamente sistemas criptográficos amplamente utilizados, como RSA, cuja segurança depende da dificuldade computacional de fatorar números grandes[cite: 6].

O presente relatório tem como objetivo relacionar a fundamentação teórica apresentada no documento *“ShorOpt.pdf”* com a implementação prática disponibilizada no notebook *“shors_algorithm.ipynb”*. Busca-se explicar como as estruturas matemáticas e físicas descritas nos slides são operacionalizadas na forma de circuitos quânticos reais utilizando ferramentas como Qiskit e IBM Quantum.

A análise aqui desenvolvida conecta três dimensões essenciais:
1.  A formulação matemática que permite reduzir fatoração ao problema da ordem;
2.  A construção do circuito quântico que estima essa ordem por meio da Estimativa de Fase Quântica (QPE);
3.  [cite_start]A interpretação dos resultados experimentais e sua combinação com algoritmos clássicos para obtenção dos fatores [cite: 9-12].

---

## 2. Metodologia de Integração Teoria-Prática

Para garantir a compreensão profunda do algoritmo, adotamos a metodologia de **"Código Anotado"**. O notebook original foi modificado para incluir docstrings e comentários que referenciam diretamente os slides da aula teórica.

As principais conexões estabelecidas no código foram:
* [cite_start]**Operadores Unitários ($U$):** A construção das portas quânticas no código foi mapeada para a definição do operador $M_a$ nos slides 46-48 [cite: 701-729].
* [cite_start]**Inicialização:** A preparação do estado $|1\rangle$ no registrador alvo foi justificada pelo "Truque do Autovetor" (Slide 36), que permite criar uma superposição uniforme [cite: 554-561].
* [cite_start]**Pós-processamento:** A função de análise clássica foi reescrita para refletir o uso de Frações Contínuas e cálculo de MDC, conforme slides 50-51 [cite: 742-767].

---

## 3. Fundamentação Teórica do Algoritmo de Shor

### 3.1. Redução da fatoração ao problema da ordem
[cite_start]O ponto central da teoria de Shor consiste em reduzir a fatoração de um número composto $N$ à determinação da ordem $r$ de um inteiro $a$ pertencente ao conjunto $Z^*_N$[cite: 15].

A função utilizada é $f(x) = a^x \pmod N$, que apresenta periodicidade $r$. Uma vez encontrado esse período, torna-se possível fatorar $N$ utilizando propriedades da aritmética modular:
$$a^r \equiv 1 \pmod N$$
$$a^r - 1 = (a^{r/2} - 1)(a^{r/2} + 1)$$

Dessa forma, os fatores de $N$ podem ser obtidos usando os máximos divisores comuns (MDC):
* $p = \text{gcd}(a^{r/2} - 1, N)$
* $q = \text{gcd}(a^{r/2} + 1, N)$

### 3.2. Definição e papel do operador $M_a$
O operador unitário $M_a$, descrito nos slides, representa a multiplicação modular:
$$M_a |x\rangle = |a \cdot x \pmod N\rangle$$

$M_a$ é uma permutação reversível e, portanto, unitária. [cite_start]Suas propriedades espectrais são fundamentais para o algoritmo, pois seus autovalores têm a forma $e^{2\pi i \cdot j/r}$, o que codifica diretamente a ordem $r$ como uma fase quântica [cite: 27-31].

### 3.3. Estimativa de Fase Quântica (QPE)
A QPE é o procedimento utilizado para extrair a fase associada aos autovalores de $M_a$. [cite_start]O material teórico descreve que a medida dos $m$ qubits superiores produz $\theta \approx y/2^m \approx j/r$[cite: 35].

O circuito implementa:
1.  Superposição inicial nos qubits de controle.
2.  Aplicações controladas das potências $M_a^{2^k}$.
3.  Transformada de Fourier Quântica inversa ($QFT^\dagger$).
4.  Medida do registrador de controle.

---

## 4. Relação entre Teoria e Implementação ($N=15$)

### 4.1. Etapas clássicas iniciais
[cite_start]A implementação começa exatamente como descrito no PDF: seleção de um valor $a \in Z^*_N$, verificação de $\text{gcd}(a, N)$ e continuidade do algoritmo apenas quando $\text{gcd}(a, N) = 1$ [cite: 52-55].

### 4.2. Implementação prática do operador $M_a$
Enquanto a teoria descreve $M_a$ de forma abstrata, o notebook o implementa com portas lógicas. [cite_start]Para o caso $N=15, a=2$, utilizamos portas **SWAP** manuais (conforme sugestão do Slide 47) para reduzir a profundidade do circuito e mitigar ruídos [cite: 58-61].

### 4.3. Resultados experimentais ($N=15$)
Assim como previsto, o notebook apresenta histogramas onde os valores mais frequentes correspondem a fases $0, 1/4, 1/2$ e $3/4$. [cite_start]Esses valores refletem $j/r$ para $r=4$, confirmando empiricamente a periodicidade da função modular [cite: 70-71].

---

## 5. O Desafio Prático: Fatorando $N=21$

Como extensão da atividade, realizamos o desafio proposto no **Slide 64**: implementar o circuito para fatorar $21$, utilizando $a=2$.

### 5.1. Configuração e Execução
* **Qubits:** 5 para o alvo ($2^4 < 21 < 2^5$) e 10 para o controle.
* **Operadores:** Utilizou-se a construção matricial genérica para as potências de $2 \pmod{21}$.

### 5.2. Análise dos Resultados
A execução no simulador `aer_simulator` gerou picos de probabilidade distintos. A análise do pico principal revelou:
1.  **Estado Medido:** Binário `0010101011` (aprox. 171 decimal).
2.  **Fase:** $\theta = 171/1024 \approx 0.1669$.
3.  **Frações Contínuas:** A fração convergente é $1/6$.
4.  **Período ($r$):** O denominador indica $r=6$.

### 5.3. Conclusão do Desafio
Com $r=6$ (par), aplicamos a fórmula clássica:
$$x = 2^{6/2} \pmod{21} = 8$$
$$MDC(8-1, 21) = \mathbf{7}$$
$$MDC(8+1, 21) = \mathbf{3}$$

O algoritmo fatorou com sucesso **$21 = 7 \times 3$**. Observamos também a natureza probabilística do Shor, onde alguns picos retornavam fatores triviais ou ordens parciais (como $r=2$), exigindo filtragem estatística.

---

## 6. Discussão e Conclusão

A implementação apresentada no notebook demonstra forte alinhamento com a formulação teórica do material de aula. Cada bloco conceitual — operador $M_a$, periodicidade, QPE e QFT — encontrou sua contraparte computacional direta.

Os resultados experimentais reproduzem fielmente os valores esperados. [cite_start]Entretanto, conforme alertado nos slides, fatorar números reais (como 2048 bits) exigiria milhões de qubits físicos e correção de erros, algo ainda inviável na tecnologia NISQ atual [cite: 84-85].

O estudo integrado permite concluir que, embora limitado pelo hardware, o algoritmo de Shor é a demonstração mais clara da superioridade assintótica da computação quântica para problemas de periodicidade oculta.

---

## 7. Referências

1.  **ShorOpt.pdf** — Slides teóricos do Algoritmo de Shor (IBM/USP).
2.  **IBM Quantum Documentation** — Shor’s Algorithm Tutorial.
3.  Shor, P. (1994). “Algorithms for quantum computation: discrete logarithms and factoring.” Proceedings 35th Annual Symposium on Foundations of Computer Science.