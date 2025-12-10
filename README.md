Relatório Acadêmico: Implementação e Análise do Algoritmo de Shor

Disciplina: Computação Quântica
Autores: Lucas de Luccas, André Hutzler, Diogo Burgierman
Data: 10 de Dezembro de 2025

⸻

1. Introdução

O algoritmo de Shor, proposto em 1994 por Peter Shor, é um dos marcos da computação quântica ao demonstrar que a fatoração de inteiros — base da segurança criptográfica RSA — pode ser realizada em tempo polinomial em computadores quânticos universais. Essa capacidade supera significativamente algoritmos clássicos, cuja complexidade permanece sub-exponencial.

Este relatório integra a fundamentação teórica apresentada no material ShorOpt.pdf com sua implementação prática no notebook shors_algorithm.ipynb, desenvolvido em Qiskit. Após reproduzir o caso clássico N = 15, o estudo se estende ao desafio prático de fatorar N = 21, analisando o comportamento probabilístico das simulações.

⸻

2. Metodologia de Integração Teoria–Prática

A abordagem adotada utilizou código anotado, conectando cada bloco de implementação do notebook a referências diretas dos slides teóricos. Essa estratégia permitiu associar cada elemento do circuito quântico aos fundamentos matemáticos do algoritmo de Shor.

Principais conexões estabelecidas

a) Operadores Unitários M_a
Com base nos slides 46–48, o notebook implementa o operador:

M_a |x\rangle = |a x \bmod N\rangle,

uma permutação unitária essencial para a construção do circuito.

b) Inicialização — “Truque do Autovetor”
Os slides 35–36 mostram que o estado |1\rangle pode ser decomposto como combinação linear de autovetores de M_a. Isso justifica sua escolha como entrada para o registrador alvo.

c) Estimativa de Fase Quântica (QPE)
A arquitetura com portas Hadamard, operações controladas e QFT inversa segue o modelo dos slides 30–43, permitindo estimar a fase:

\theta = \frac{j}{r},

associada aos autovalores de M_a.

d) Pós-processamento Clássico
Os slides 50–51 fundamentam a recuperação da ordem r através de frações contínuas, seguida dos cálculos:

\gcd(a^{r/2}-1, N), \quad \gcd(a^{r/2}+1, N),

que retornam fatores não triviais de N.

⸻

3. Fundamentação Teórica e Implementação (N = 15)

3.1. Redução do Problema

O algoritmo não fatora N diretamente; ele determina a ordem r da função:

f(x) = a^x \bmod N.

Se r é par e atende às condições adequadas, os fatores de N advêm de:

p = \gcd(a^{r/2} - 1, N), \quad q = \gcd(a^{r/2} + 1, N).

3.2. Otimização do Circuito

Para N = 15 e a = 2, o ciclo modular:

1 → 2 → 4 → 8 → 1

revela período r = 4.

No notebook, operadores como M_2 e M_4 foram implementados via portas SWAP, reduzindo a profundidade do circuito conforme sugerido no slide 47 — prática importante para execução em hardware real.

⸻

4. Desafio Prático: Fatorando N = 21

4.1. Parâmetros do Circuito

Para fatorar 21 com a = 2, utilizamos:
	•	5 qubits de alvo, pois 2^4 < 21 < 2^5;
	•	10 qubits de controle, necessários para precisão da QPE;
	•	Implementação genérica dos operadores M_a^{2^k}, devido à estrutura modular mais complexa.

⸻

4.2. Resultados e Análise do Histograma

A simulação no aer_simulator produziu picos significativos no histograma. O estado mais provável foi:

0010101011

Convertido para decimal:

171

A fase aproximada é:

\theta = \frac{171}{1024} \approx 0.1669.

Frações contínuas fornecem a aproximação:

\theta \approx \frac{1}{6},

indicando que:

r = 6.


⸻

4.3. Validação Clássica

Com r = 6:

a^{r/2} = 2^3 = 8.

Assim:

\gcd(8 - 1, 21) = \gcd(7, 21) = 7,  
\gcd(8 + 1, 21) = \gcd(9, 21) = 3.

Logo:

21 = 3 \times 7.

A análise confirma a eficácia estatística do algoritmo, embora execuções individuais possam produzir r parcial (2 ou 3), reforçando a necessidade de filtragem probabilística.

⸻

5. Discussão e Limitações

A implementação apresentou consistência com os princípios teóricos estudados. Entretanto, limitações práticas permanecem:

Ruído e Decoerência

Execuções em hardware real apresentam dispersão no histograma, ocasionada por ruído de portas e acoplamentos parasitas.

Escalabilidade

Embora funcional em exemplos pequenos, a fatoração de números RSA (2048 bits) exigiria:
	•	milhões de qubits físicos;
	•	correção de erros em larga escala;
	•	profundidade de circuito muito além das capacidades NISQ atuais.

Assim, aplicações reais permanecem distantes, apesar do poder teórico exemplar demonstrado pelo algoritmo.

⸻

6. Referências
	1.	ShorOpt.pdf — Material de aula: Algoritmo de Shor e Implementação em Circuitos Quânticos (Prof. Bryan Kano & Prof. Routo Terada).
	2.	Notebook: shors_algorithm.ipynb — Implementação prática em Qiskit.
	3.	Shor, P. W. (1994). Algorithms for quantum computation: discrete logarithms and factoring. Proceedings of the 35th Annual Symposium on Foundations of Computer Science.

