# Relatório Acadêmico: Implementação e Análise do Algoritmo de Shor

**Disciplina:** Computação Quântica  
**Autores:** Lucas de Luccas, André Hutzler, Diogo Burgierman e [Seu Nome]  
**Data:** 10 de Dezembro de 2025

---

## 1. Introdução

[cite_start]O algoritmo de Shor, proposto em 1994 por Peter Shor, representa um marco na computação quântica ao oferecer uma solução em tempo polinomial para a fatoração de inteiros — um problema considerado intratável para computadores clássicos e base da segurança criptográfica RSA [cite: 5-6].

Este projeto tem como objetivo conectar a fundamentação teórica (baseada no material de aula `ShorOpt.pdf`) com a implementação prática utilizando o SDK Qiskit. Além da reprodução do caso clássico ($N=15$), este relatório documenta a resolução do **Desafio Prático ($N=21$)**, analisando os resultados probabilísticos obtidos via simulação.

## 2. Metodologia de Integração Teoria-Prática

Para garantir a compreensão profunda do algoritmo, adotamos a metodologia de **"Código Anotado"**. O notebook original (`shors_algorithm.ipynb`) foi modificado para incluir docstrings e comentários que referenciam diretamente os slides da aula teórica.

As principais conexões estabelecidas foram:

* [cite_start]**Operadores Unitários ($U$):** A construção das portas quânticas no código foi mapeada para a definição do operador $M_a$ nos slides 46-48 [cite: 701-729].
* [cite_start]**Inicialização:** A preparação do estado $|1\rangle$ no registrador alvo foi justificada pelo "Truque do Autovetor" (Slide 36), que permite criar uma superposição uniforme de todos os autovetores [cite: 554-561].
* [cite_start]**Estimativa de Fase (QPE):** A construção do circuito com portas Hadamard e QFT Inversa foi alinhada aos diagramas dos slides 30-43 [cite: 472-512, 647-656].
* [cite_start]**Pós-processamento:** A função de análise clássica foi reescrita para refletir o uso de Frações Contínuas e cálculo de MDC, conforme slides 50-51 [cite: 742-767].

## 3. Fundamentação Teórica e Implementação ($N=15$)

### 3.1. Redução do Problema
O algoritmo não fatora $N$ diretamente; ele encontra o **período (ordem) $r$** da função modular $f(x) = a^x \pmod N$. [cite_start]Se $r$ for par, os fatores de $N$ podem ser derivados via $MDC(a^{r/2} \pm 1, N)$ [cite: 398-406].

### 3.2. Otimização de Circuito (Slide 47)
Para o caso $N=15$ e $a=2$, a teoria prevê a sequência $1 \to 2 \to 4 \to 8 \to 1$ ($r=4$).
[cite_start]No código, em vez de uma matriz unitária genérica, implementamos os operadores $M_2$ e $M_4$ manualmente utilizando portas **SWAP**, conforme sugerido no Slide 47. Isso reduz drasticamente a profundidade do circuito, mitigando erros em hardware real [cite: 701-710].

## 4. O Desafio: Fatorando $N=21$

Como extensão da atividade proposta (Slide 64), implementamos o circuito para fatorar **21**, utilizando **$a=2$**.

### 4.1. Configuração do Circuito
Diferente do caso $N=15$, a fatoração de 21 exige mais recursos:
* **Qubits de Alvo:** 5 (pois $2^4 < 21 < 2^5$).
* **Qubits de Controle:** 10 (para precisão suficiente na estimativa de fase).
* **Operadores:** Utilizou-se a construção matricial genérica para as potências de $2 \pmod{21}$.

### 4.2. Análise dos Resultados (Histograma)

A execução no simulador `aer_simulator` gerou a distribuição de probabilidades apresentada abaixo:

![Histograma N=21](image_ebccc0.png)

**Interpretação dos Picos:**
Observando o histograma, destacam-se picos de contagem que correspondem às fases construtivas da interferência quântica.

1.  **Leitura do Pico Principal:** O gráfico mostra uma contagem alta para o estado `0010101011` (aprox. decimal 171).
2.  **Cálculo da Fase:** $\theta = \frac{171}{1024} \approx 0.1669$.
3.  **Frações Contínuas:** A fração mais próxima é $\frac{1}{6}$.
4.  **Determinação do Período:** O denominador indica que a ordem é **$r = 6$**.

### 4.3. Validação Clássica
Com $r=6$ (que é par), aplicamos a fórmula de fatoração descrita na teoria:
1.  Calcular $x = a^{r/2} \pmod N \Rightarrow 2^{6/2} \pmod{21} = 2^3 = 8$.
2.  Calcular Fatores:
    * $MDC(8 - 1, 21) = MDC(7, 21) = \mathbf{7}$
    * $MDC(8 + 1, 21) = MDC(9, 21) = \mathbf{3}$

**Conclusão do Desafio:** O algoritmo fatorou com sucesso $21 = 7 \times 3$. Observou-se na prática a natureza probabilística do algoritmo, onde alguns picos retornam fatores triviais ou ordens parciais (como $r=2$ ou $r=3$), exigindo filtragem estatística dos resultados para encontrar o período correto ($r=6$).

## 5. Discussão e Limitações

A implementação confirmou o alinhamento entre a teoria (PDF) e a prática (Notebook). No entanto, ressalta-se as limitações de hardware discutidas no Slide 52:
* [cite_start]**Ruído:** Em hardware real (como `ibm_marrakesh`), o histograma apresentaria "vazamento" de probabilidade devido à decoerência e erros de porta [cite: 768-782].
* [cite_start]**Escalabilidade:** Fatorar números RSA reais (2048 bits) exigiria milhões de qubits e correção de erros robusta, algo ainda distante da tecnologia NISQ atual [cite: 800-814].

## 6. Referências

1.  **ShorOpt.pdf** — Material de Aula: Análise Teórica e Implementação em Circuitos Quânticos (Prof. Bryan Kano & Prof. Routo Terada).
2.  **Notebook** — `shors_algorithm.ipynb` (IBM Quantum / Qiskit Tutorials).
3.  Shor, P. W. (1994). "Algorithms for quantum computation: discrete logarithms and factoring".