# Relatório comparativo – Módulo de Relatórios x Módulo BIS (Avançado)

Este documento resume o que o pacote **BIS (Business Intelligence)** já implementa em relação ao módulo de **Relatórios padrão** do i-Educar, e o que ainda falta para uma cobertura mais completa / integrada.

> Referências:
> - Módulo BIS: código em `packages/serventec/i-educar-bis-package/` e documentação em `README.md` e `ANALISE_COMPOSER_E_TESTES.md`.
> - Módulo de Relatórios padrão: repositório oficial `portabilis/i-educar-reports-package` (linkado no `README.md` do i-Educar).

---

## 1. O que o BIS já implementa

### 1.1. Estrutura de pacote e integração

- Pacote isolado em `packages/serventec/i-educar-bis-package`.
- `BisProvider`:
  - Faz `mergeConfig` de `config/bis.php`.
  - Registra views em `resources/views` com namespace `bis`.
  - Carrega rotas `routes/web.php`.
  - Registra migrations próprias (menu BI).
  - Publica assets CSS (`bi-dashboard.css`, `bi-print.css`) via tag `bis-assets`.
  - Registra middleware `ieducar.bi.menu` para controlar visibilidade do menu BI.
  - Registra componentes Blade (`<x-bi-powered>`, `<x-bi-print-wrapper>`).

### 1.2. Rotas e menus

- Prefixo `/bis` com middleware padrão do i-Educar (`web`, `auth`, `ieducar.navigation`, `ieducar.footer`, etc.).
- Rotas principais:
  - `GET /bis` → `DashboardController@index` (`bis.dashboard`).
  - `GET /bis/relatorios` → `ReportController@index` (`bis.reports`).
  - `GET /bis/matriculas` → `MatriculasController@index` (`bis.matriculas.index`).
  - `GET /bis/turmas` → `TurmasController@index` (`bis.turmas.index`).
  - `GET /bis/{theme}` → `ThemeController@show` (`bis.theme`) para temas:
    - `lancamentos`
    - `indicadores`
    - `inclusao-diversidade`
    - `busca-ativa`
    - `educacenso`

### 1.3. Serviços e relatórios já existentes

- `BiDashboardService`:
  - Consolida **resumo geral** por ano letivo:
    - Matrículas por curso.
    - Turmas por escola.
    - Evolução temporal (gráfico de linha).
  - Funções:
    - `getAnosLetivos()`.
    - `getSummary($ano)`: dados + gráficos (`Chart.js` via `consoletvs/charts`).
- `BiMatriculasReportsService`:
  - Lista anos letivos, instituições, escolas, cursos, turnos.
  - Filtros:
    - Ano (ou todos).
    - Situação de enturmação (enturmados / não enturmados).
    - Instituição, escola, curso.
    - Modalidade (1–4).
    - Dependência (sim/não).
    - Turno.
  - `getAllReports($filters)`: retorna matriz de relatórios analíticos com título, totalizadores e, em alguns casos, gráficos.
- `BiTurmasReportsService`:
  - Relatórios/dashboards focados em turmas (por escola, curso, turno, etc.).
- `BiChartsService`:
  - Criação dos objetos de gráfico compatíveis com `consoletvs/charts` e `Chart.js`.

### 1.4. Views / UI

- Views organizadas por tema:
  - `resources/views/dashboard/index.blade.php` – painel geral.
  - `resources/views/matriculas/index.blade.php` – relatórios + filtros de matrículas.
  - `resources/views/turmas/index.blade.php` – relatórios + filtros de turmas.
  - `resources/views/reports/index.blade.php` – hub de relatórios do BI.
  - `resources/views/themes/show.blade.php` – temas específicos (lançamentos, indicadores, inclusão e diversidade, busca ativa, Educacenso).
- Componentes de impressão:
  - `bi-print-header`, `bi-print-footer`, `bi-print-wrapper`.
  - CSS dedicado para impressão (`bi-print.css`).

### 1.5. Permissões e processos

- Uso de `BisProcess` para mapear constantes de processo do BI:
  - `menuBi()`, `dashboard()`, `matriculas()`, `turmas()`, `lancamentos()`, `indicadores()`, etc.
- Gate do Laravel:
  - `Gate::allows('view', BisProcess::matriculas())` e similares para proteger cada rota.

### 1.6. Testes

- Testes unitários e de feature:
  - `BisProcessTest` – valida constantes e helpers de processo.
  - `BiDashboardServiceTest` – testa lógica do serviço principal (exige banco PostgreSQL).
  - `DashboardControllerTest` – garante que a rota `/bis` está cadastrada e responde.
- Integração com PHPUnit/Pest via `phpunit.package.xml` e `tests/Pest.php`.

---

## 2. Comparação conceitual com o módulo de Relatórios padrão

> O módulo de Relatórios padrão (portabilis/i-educar-reports-package) é focado em **relatórios operacionais** (PDF, impressos, saídas específicas do sistema), enquanto o BIS é focado em **análises agregadas e dashboards**.

### 2.1. Relatórios x Dashboards

- **Relatórios padrão**:
  - Saídas estáticas (geralmente PDFs, listagens).
  - Foco em **registro** (boletins, declarações, históricos, relatórios de matrícula, frequências).
  - Muito orientados a documentos oficiais e comprovações.
- **BIS**:
  - Foco em **painéis analíticos** (gráficos, tabelas agregadas, indicadores).
  - Visões cruzadas (matrículas x curso, turmas x escola, evasão, inclusão, etc.).
  - Suporte a impressão, mas com layout orientado a dashboards.

### 2.2. Cobertura de temas

- Temas que ambos tocam (mas com enfoques diferentes):
  - Matrículas.
  - Turmas.
  - Lançamentos (notas/faltas).
  - Indicadores (aprovação, reprovação, evasão).
  - Inclusão/diversidade (AEE, deficiência, cor/raça).
  - Educacenso/INEP (cobertura por escola/curso).
- Temas mais fortes no **relatórios padrão**:
  - Boletins individuais.
  - Atas, diários, declarações.
  - Relatórios de frequências detalhados por aluno.
- Temas mais fortes no **BIS**:
  - Vistas consolidadas por rede/escola/curso/ano.
  - Evolução temporal e comparações entre períodos.
  - Integração com componentes de gráfico (Chart.js).

---

## 3. O que falta / oportunidades de evolução no BIS

### 3.1. Paridade com relatórios padrão

Itens que ainda não estão cobertos no BIS, mas poderiam ser agregados como “visões analíticas”:

- **Relatórios de frequência**:
  - Hoje o BIS apresenta indicadores, mas poderia ter:
    - Heatmaps de frequência por turma e período.
    - Comparativos entre escolas/turnos.
- **Boletins agregados**:
  - Dashboards que sumarizem notas por turma/curso, em vez de apenas relatórios impressos.
- **Relatórios específicos do Educacenso**:
  - Indicadores de consistência da base (quantos alunos sem dados obrigatórios, etc.).

### 3.2. Integração mais profunda com o módulo de relatórios

Pontos a evoluir:

- Links diretos:
  - De cartões/indicadores do BI para relatórios operacionais (ex.: clicar em um indicador de evasão e abrir relatório detalhado de alunos evadidos).
- Filtros compartilhados:
  - Criar uma camada comum de filtros (ano, escola, curso) entre BIS e relatórios padrão, para que o usuário não precise refazer seleções.
- Exportações adicionais:
  - Exportar tabelas do BI em CSV/XLSX com o mesmo layout das saídas do módulo de relatórios quando fizer sentido.

### 3.3. Configurabilidade e temas

- Tornar os **temas** (`lancamentos`, `indicadores`, `inclusao-diversidade`, `busca-ativa`, `educacenso`) mais configuráveis:
  - Habilitar/desabilitar temas por instituição ou por tipo de usuário.
  - Permitir que os temas sejam estendidos por outros pacotes (ex.: Gestão Presente).

### 3.4. Documentação funcional comparativa

Ainda falta um documento que mapeie, tela a tela, algo como:

- Relatório X do módulo padrão ↔ Widget/Y do BIS (ou “não existe no BIS”).

Esse mapeamento ajudaria:

- Usuários finais a entender quando usar o BI ou o relatório tradicional.
- Desenvolvedores a enxergar lacunas e oportunidades de novos gráficos.

---

## 4. Resumo

O pacote **BIS** já entrega:

- Estrutura de pacote Laravel integrada ao i-Educar.
- Menu próprio, permissões e middleware dedicados.
- Dashboards e relatórios analíticos para:
  - Matrículas, turmas, lançamentos, indicadores, inclusão/diversidade, busca ativa, Educacenso.
- Componentes de impressão e estilização própria.
- Suite de testes integrada ao padrão `packages` (Pest/PHPUnit).

Faltam, principalmente:

- Mapear e aproximar a experiência entre **Relatórios padrão** e **BIS** (links, filtros, exportações).
- Expandir o BIS com mais visões inspiradas nos relatórios já existentes (especialmente frequência e documentos oficiais em formato analítico).
- Documentar essa matriz de equivalência/integração para orientar tanto usuários quanto desenvolvedores.

