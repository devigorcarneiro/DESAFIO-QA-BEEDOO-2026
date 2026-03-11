# DESAFIO-QA-BEEDOO-2026

Repositório destinado à entrega do desafio técnico para a vaga de QA na Beedoo. Este documento detalha a análise exploratória, as decisões, os cenários levantados e os bugs estruturais encontrados na aplicação disponibilizada.

## 1. Análise inicial da aplicação

**Objetivo da aplicação:**
A aplicação tem como objetivo fornecer um painel administrativo simples e direto para o cadastro e a listagem de cursos, permitindo gestão do catálogo.

**Principais fluxos disponíveis:**
1. **Cadastro de cursos:** Formulário para inserção de novos dados (nome, descrição, datas, vagas, imagem e tipo).
2. **Listagem de cursos:** Visualização em interface de *cards* dos cursos previamente cadastrados na base.
3. **Exclusão de cursos:** Ação de exclusão a partir do card na listagem.

**Pontos críticos identificados:**
Com base na aplicação analisada, os pontos mais sensíveis mapeados para teste foram:
* **Validação de inputs:** O front-end precisa garantir a integridade dos dados (campos nulos, lógicas de data e valores numéricos) antes de tentar persistir informações no banco de dados.
* **Sincronia de estado:** A interface gráfica deve refletir o real estado da base de dados, especialmente em operações destrutivas como exclusões.

---

## 2. Decisões tomadas e raciocínio analítico

Durante a elaboração dos testes, optei por focar a cobertura técnica nos **fluxos de exceção (cenários negativos)**. Em sistemas de cadastro, o "caminho feliz" costuma funcionar, mas a verdadeira qualidade da aplicação é testada quando o usuário insere dados inesperados. 

Meu raciocínio foi focado na **integridade de dados**:
* **Uso de Gherkin:** Estruturei os casos de teste utilizando a sintaxe Gherkin para manter o foco na regra de negócio e facilitar futuras implementações de testes automatizados (ex: Cypress).
* **Foco em UX e confiabilidade:** Analisei como o sistema lida com o feedback ao usuário. Identifiquei que a interface reage de forma "otimista" a cliques, exibindo mensagens de sucesso sem verificar se a mutação de dados realmente ocorreu no backend, o que gera falsos positivos graves.

---

## 3. Cenários e casos de teste

Os cenários de teste foram documentados detalhadamente em formato estruturado, cobrindo as falhas de validação mapeadas na exploração.

📊 **[Acessar planilha de casos de teste (Google Sheets)](https://docs.google.com/spreadsheets/d/15LcWJ1yWQMQ8OiVVQdVJ5vpELu5JkHHQwEz_awZEvIU/edit?usp=sharing)**

---

## 4. Evidências da execução

Todas as evidências em formato de imagem (capturas de tela) comprovando o comportamento atual do sistema e a falha nos testes estão armazenadas no Google Drive.

📁 **[Acessar evidências de teste (Google Drive)](https://drive.google.com/drive/folders/1J4YH1RPU-ceLHOn39THnxoLipptFnPhR?usp=sharing)**

---

## 5. Relatório de bugs encontrados

Abaixo estão detalhados os problemas críticos encontrados durante a execução dos testes:

### Bug 001: Ausência de validação de campos obrigatórios
* **Severidade / Impacto:** Alta. (Permite a criação de registros nulos, corrompendo a integridade da listagem).
* **Passos para reproduzir:**
  1. Acessar a aba "Cadastrar curso".
  2. Deixar todos os campos do formulário totalmente em branco.
  3. Clicar em "Cadastrar Curso".
  4. Navegar até a aba "Listar cursos".
* **Resultado Atual:** O sistema não exige obrigatoriedade, cadastra o curso e renderiza um card vazio na listagem.
* **Resultado Esperado:** O sistema deve bloquear a submissão e exibir mensagens de alerta nos campos obrigatórios.

### Bug 002: Inconsistência cronológica (Data Fim anterior à Data Início)
* **Severidade / Impacto:** Média. (Quebra de regra de negócio, gerando inconsistência no período do curso).
* **Passos para reproduzir:**
  1. Acessar a aba "Cadastrar curso".
  2. Preencher os dados de texto.
  3. Inserir uma "Data de início" (ex: 17/03/2026).
  4. Inserir uma "Data de fim" cronologicamente anterior (ex: 15/01/2026).
  5. Submeter o formulário.
* **Resultado Atual:** O sistema aceita a viagem no tempo e cadastra o curso com o período inválido.
* **Resultado Esperado:** Bloquear a submissão informando que a data de término deve ser igual ou posterior à data de início.

### Bug 003: Permissão de valor negativo no número de vagas
* **Severidade / Impacto:** Média. (Afeta regras de negócio e lógicas futuras de matrícula).
* **Passos para reproduzir:**
  1. Acessar a aba "Cadastrar curso".
  2. Preencher as informações.
  3. No campo "Número de vagas", inserir um valor negativo (ex: `-10`).
  4. Submeter o formulário.
* **Resultado Atual:** O sistema não faz validação de limite mínimo (`min >= "0"`) e cadastra o curso com vagas negativas.
* **Resultado Esperado:** Bloquear o cadastro informando que a quantidade de vagas deve ser maior ou igual a zero.

### Bug 004: Falso positivo na exclusão de cursos (State Management)
* **Severidade / Impacto:** Alta. (Falha grave de UX e confiabilidade, o sistema informa sucesso em operação que não ocorreu).
* **Passos para reproduzir:**
  1. Acessar a aba "Listar cursos".
  2. Clicar no botão "Excluir curso" em qualquer card existente.
* **Resultado Atual:** O sistema exibe um Toast de "Curso excluído com sucesso!", mas o card não é removido do DOM/Listagem. Cliques adicionais apenas empilham a notificação sem excluir o dado.
* **Resultado Esperado:** O sistema deve realizar a exclusão real (removendo do banco), atualizar o estado da interface gráfica removendo o card, e apenas exibir a notificação se a operação for bem-sucedida.
