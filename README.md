# i-Educar BIS (Business Intelligence)

Módulo de Business Intelligence para geração de dashboards e relatórios analíticos no [i-Educar](https://github.com/portabilis/i-educar).

## Repositórios

Código do pacote (Power BI / BIS) — organização **i-Educar**:

- **SSH:** `git@github.com:i-Educar/i-educar-powerbi-package.git`
- **HTTPS:** https://github.com/i-Educar/i-educar-powerbi-package

Espelhos / origem histórica:

- Serventec: https://github.com/serventecieducar/i-educar-powerbi-package

Projeto i-Educar (onde o pacote é utilizado):

- **HTTPS:** https://github.com/portabilis/i-educar

**Compatibilidade:** i-Educar **2.12** (PHP ≥ 8.3). Pasta local no monorepo: `packages/serventec/i-educar-bis-package` (composer: `serventec/i-educar-bis-package`).

## Dependências

- [Chart.js](https://www.chartjs.org/) via `consoletvs/charts` para gráficos
- PHP >= 8.3
- Laravel 11+ (via projeto i-Educar 2.12)

## Estrutura do pacote

```
packages/serventec/i-educar-bis-package/
├── config/
│   └── bis.php              # Configuração (chart_library, cache, etc.)
├── database/migrations/      # Menu BI e permissões
├── resources/
│   ├── assets/css/
│   │   └── bi-print.css      # Estilos para impressão
│   └── views/                # Views Blade (dashboard, matrículas, turmas, etc.)
├── routes/web.php            # Rotas em /bis/*
├── src/
│   ├── BisProcess.php        # Constantes e helpers do BI
│   ├── Http/Controllers/     # Controllers
│   ├── Providers/BisProvider.php
│   └── Services/             # BiDashboardService, BiChartsService, etc.
├── tests/                    # Testes unitários e de feature
└── docs/                     # Documentação técnica
```

## Instalação

### Ritual automatizado (recomendado)

Na raiz do i-Educar:

```bash
# Clone (se ainda não estiver em packages/serventec/i-educar-bis-package)
git clone https://github.com/i-Educar/i-educar-powerbi-package.git packages/serventec/i-educar-bis-package

# Script do monorepo (composer path + migrate + assets)
bash scripts/instalar-bis

# Ou via Artisan
php artisan bis:instalar
```

No `composer.json` do i-Educar (já presente no monorepo Laguna):

- `require`: `"serventec/i-educar-bis-package": "@dev"`
- `repositories`: path `packages/serventec/i-educar-bis-package`
- `autoload.psr-4`: `"iEducar\\Packages\\Bis\\": "packages/serventec/i-educar-bis-package/src/"`

Em `config/tenants.local.php`, habilite o pacote nos tenants desejados:

```php
'serventec/i-educar-bis-package' => ['laguna', 'tenant1', 'tenant2', 'pgsql'],
```

### Via Plug and Play (desenvolvimento)

```bash
git clone https://github.com/i-Educar/i-educar-powerbi-package.git packages/serventec/i-educar-bis-package
composer plug-and-play
php artisan migrate
php artisan vendor:publish --tag=bis-assets --force
```

### Via Composer (VCS)

```json
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/i-Educar/i-educar-powerbi-package.git"
    }
  ],
  "require": {
    "serventec/i-educar-bis-package": "@dev"
  }
}
```

```bash
composer update serventec/i-educar-bis-package
php artisan migrate
php artisan vendor:publish --tag=bis-assets --force
```

Limpe caches se necessário: `php artisan optimize:clear`.

**Observação:** se o i-Educar possuir as constantes `Process::BI_*` e `Process::MENU_BI` em `App\Process`, o pacote usará esses mesmos IDs. Caso contrário, o `BisProcess` aplica valores próprios e a migration continua funcionando em instalações novas.

## Estrutura de menus

O módulo adiciona o menu **BI** em **Escola**, com submenus por tema:

- **Matrículas** – Dashboard de matrículas por curso, escola, situação e mais
- **Turmas** – Distribuição de turmas por escola, curso e turno
- **Lançamentos** – Notas e faltas por etapa
- **Indicadores** – Evasão, aprovação, reprovação, reclassificação, abandono
- **Inclusão e Diversidade** – Matrículas por deficiência, cor/raça, gênero e AEE
- **Busca Ativa** – Casos de evasão e resultado do programa
- **Educacenso/INEP** – Cobertura INEP e registros do Censo Escolar

As permissões por submenu podem ser ajustadas em **Configurações > Permissões > Tipos de usuário**.

## Configuração

Arquivo `config/bis.php` (ou variáveis de ambiente):

| Variável              | Descrição                                   | Padrão   |
|-----------------------|---------------------------------------------|----------|
| `BIS_CHART_LIBRARY`   | Biblioteca de gráficos (chartjs, highcharts, etc.) | chartjs |
| `BIS_CACHE_TTL`       | Tempo de cache em segundos (0 = desabilitado)    | 300      |
| `BIS_POWERED_BY_AUTHOR` | Crédito de autoria exibido nos dashboards   | JaderGabriel |

## Testes

**Do projeto raiz (i-Educar):**

```bash
# Suite BIS completa
php vendor/bin/phpunit --testsuite=BIS

# Testes que não exigem banco
php vendor/bin/phpunit packages/serventec/i-educar-bis-package/tests/Unit/BisProcessTest.php
php vendor/bin/phpunit packages/serventec/i-educar-bis-package/tests/Feature/DashboardControllerTest.php
```

**Do diretório do pacote:**

```bash
cd packages/serventec/i-educar-bis-package
composer test
```

Os testes `BiDashboardServiceTest` e similares exigem PostgreSQL. Consulte `docs/ANALISE_COMPOSER_E_TESTES.md` para detalhes.

## Fluxo de trabalho

Todo commit, push e criação de branch de melhorias devem ocorrer em `packages/serventec/i-educar-bis-package`, mantendo o repositório do BI separado do i-Educar principal.

## Impressão

Os dashboards e gráficos suportam impressão com cabeçalho e rodapé padrões (configuráveis em Configurações gerais).

## Capturas de tela

- Dashboard BI: `docs/screenshots/bis-dashboard.png`
- Matrículas BI: `docs/screenshots/bis-matriculas.png`
- Turmas BI: `docs/screenshots/bis-turmas.png`


## FAQ

Perguntas frequentes: [FAQ i-Educar](https://github.com/portabilis/i-educar-website/blob/master/docs/faq.md).

---

Powered by [Serventec](https://serventecassessoria.com.br).
Powered by [JaderGabriel](https://t.me/JaderGabriel).
