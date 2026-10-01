# Schedule-Manager
Este projeto é um otimizador matemático para a alocação de horários de aulas universitárias. Ele utiliza a biblioteca **Google OR-Tools** (mais especificamente o solver **CP-SAT**) para resolver o complexo quebra-cabeça de agendamento de disciplinas, professores e turnos, respeitando rigorosas regras pedagógicas e preferências institucionais.

## Principais Funcionalidades

A inteligência por trás deste projeto não se limita a apenas evitar choques de horário. O algoritmo atua com base em **Hard Constraints** (regras inquebráveis) e **Soft Constraints** (regras otimizadas por função objetivo):

* **Encaixe em Blocos Perfeitos (Anti-Fragmentação):** Aulas são alocadas estritamente em pares (2h consecutivas) ou trincas (3h consecutivas), garantindo que não fiquem "buracos" de 1 hora soltos na grade.
* **Slot Fixo:** Se uma disciplina ocorre duas ou três vezes na semana, ela será alocada **exatamente no mesmo horário** em todos esses dias (ex: sempre às Terças e Quintas, na T2 e T3).
* **Otimização da Agenda do Professor:** O solver minimiza matematicamente o número de dias que um professor precisa ir à universidade. Se ele leciona múltiplas disciplinas, o modelo tentará agrupá-las nos mesmos dias.
* **Alternância de Turnos entre Semestres:** Semestres ímpares (ex: 7º e 9º) são priorizados pela manhã, enquanto semestres pares (ex: 6º, 8º e 10º) são priorizados à tarde. Isso permite que alunos peguem dependências sem choques de horário.
* **Transbordamento Inteligente:** Se um turno preferencial atingir o limite de capacidade, o sistema não quebra (Infeasible); ele transborda o excedente para o turno alternativo organicamente.
* **Horários "Blocados" (Pré-Fixados):** Suporte nativo para travar disciplinas em horários específicos previamente acordados, lendo diretamente da planilha.

## Como Utilizar

### 1. Pré-requisitos
O projeto roda em **Python** e recomenda-se o uso do **Jupyter Notebook**.
Instale as dependências executando:
```bash
pip install ortools pandas openpyxl
```

### 2. Preparando os Dados (semestre.xlsx)
O modelo consome os dados de um arquivo Excel chamado `semestre.xlsx`. 
Ele espera uma aba chamada `disciplinas` contendo as seguintes colunas obrigatórias:
* `periodo`: Inteiro representando o semestre da disciplina (ex: 6, 8, 9).
* `disciplina`: Nome da disciplina (Textos contendo "optativa" viram curingas de horário e turmas "controle" forçam uso de Trincas).
* `professor`: Nome do professor (usado para evitar choques e agrupar dias). Use "nda" para ignorar.
* `carga horária`: Deve ser `30` (1 par de aulas), `60` (2 pares) ou `90` (2 trincas ou 3 pares).
* `blocado`: Para pré-alocar a aula, insira no formato `[Dia][Turno][Horários]` separados por ponto e vírgula. Ex: `2t45;4t45` trava a aula na Segunda (2) e Quarta (4) na tarde 4 e 5. Se livre, deixe como `não` ou `nan`.

### 3. Executando o Solver
Abra o arquivo `horarios.ipynb` e rode as células. 
O solver começará a cruzar os dados, criar as variáveis e varrer milhões de combinações possíveis. Em poucos segundos (limitado a 120s por padrão), ele retornará:
* `OPTIMAL` ou `FEASIBLE`: O modelo encontrou uma grade com sucesso.
* `INFEASIBLE`: As restrições de horários e professores entraram num conflito matemático impossível de ser resolvido. (Geralmente indica falta de espaço num turno muito requisitado).

O resultado otimizado imprimirá também a pontuação da função objetivo, indicando o grau de agrupamento perfeito encontrado.

---
**Dica de Otimização:** O algoritmo prioriza que o resultado final não só seja matematicamente possível, mas que ofereça a melhor "qualidade de vida" possível para alunos e professores!
