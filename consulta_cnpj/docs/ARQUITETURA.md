# Arquitetura - Consulta CNPJ

## 1. Diretriz

O projeto deve utilizar principios de:

- Clean Architecture;
- SOLID;
- separacao de responsabilidades;
- Domain-Driven Design de forma pragmatica;
- Dependency Inversion para infraestrutura;
- Strategy Pattern para fontes externas;
- ports and adapters para entrada, saida e atualizacao.

DDD deve tornar explicitas as regras de validacao e consulta, sem criar complexidade desnecessaria.

## 2. Contexto delimitado

O MVP possui um bounded context principal:

**Validacao Cadastral de Fornecedores Datasul**

Responsabilidades:

- extrair ou receber fornecedores do Datasul;
- normalizar CNPJs;
- validar CNPJ;
- consultar situacao cadastral na Receita Federal ou fonte homologada;
- padronizar resposta;
- comparar dados Receita x Datasul;
- gerar relatorio de inconsistencias;
- gerar saida de atualizacao apos aprovacao;
- registrar resultado e pendencias.

APIs externas, Datasul, arquivos, planilhas, programas ABL, interface grafica e console sao detalhes externos ao dominio.

## 3. Camadas

```text
Interface (GUI / CLI / Scripts)
        |
        v
Application / Use Cases
        |
        v
Domain
        ^
        |
Infrastructure
```

### Domain

Nao deve depender de:

- requests/httpx;
- pandas/openpyxl;
- Tkinter/CustomTkinter;
- sistema operacional;
- formato bruto de API externa;
- logging concreto.

Objetos sugeridos:

- `Cnpj`;
- `Empresa`;
- `FornecedorDatasul`;
- `Endereco`;
- `Cnae`;
- `SituacaoCadastral`;
- `ResultadoValidacao`;
- `InconsistenciaCadastral`;
- `PendenciaValidacao`;
- `AprovacaoAtualizacao`;
- `StatusValidacao`.

### Application

Casos de uso sugeridos:

- `ExtrairFornecedoresDatasul`;
- `ValidarFornecedores`;
- `ValidarCnpj`;
- `CompararCadastroFornecedor`;
- `GerarRelatorioInconsistencias`;
- `GerarSaidaAtualizacaoDatasul`;
- `RegistrarResumoExecucao`.

A camada de aplicacao orquestra dominio e portas de infraestrutura.

### Infrastructure

Implementacoes:

- clientes HTTP de fontes externas;
- cliente API Datasul;
- leitores de CSV/XLSX;
- exportadores de resultado;
- adaptadores de atualizacao Datasul por CSV, API ou ABL;
- cache, se adotado;
- logging;
- configuracao;
- empacotamento Windows.

### Interface

Adaptadores possiveis:

- CLI;
- GUI Windows;
- scripts internos;
- API futura.

Todas as interfaces devem usar os mesmos casos de uso.

## 4. Componentes substituiveis

Cada etapa que conversa com tecnologia externa deve ser representada por uma porta:

```text
FornecedorInput
CadastroCnpjProvider
InconsistencyOutput
ApprovalInput
UpdateOutput
```

Implementacoes concretas devem ficar na infraestrutura. Trocar API por arquivo, CSV por API ou CSV por ABL nao deve alterar dominio nem regras de comparacao.

Detalhes e contratos estao em `COMPONENTES_SUBSTITUIVEIS.md`.

## 5. Estrategia para fontes externas

Contrato conceitual:

```python
class FonteConsultaCnpj:
    def consultar(self, cnpj: Cnpj) -> ResultadoFonte:
        ...
```

Implementacoes possiveis:

- `FonteReceitaWs`;
- `FonteBrasilApi`;
- `FonteSerpro`;
- `FonteMock`.

Um registry/factory deve selecionar a fonte configurada.

## 6. Estrategia para Datasul

Contratos conceituais:

```python
class FornecedorInput:
    def listar_fornecedores(self) -> list[FornecedorDatasul]:
        ...

class UpdateOutput:
    def enviar(self, atualizacoes: list[AprovacaoAtualizacao]) -> ResultadoAtualizacao:
        ...
```

No MVP, a entrada pode ser arquivo exportado do Datasul enquanto a API nao estiver definida. A mesma porta deve suportar posteriormente API Datasul sem alterar o caso de uso.

Para atualizacao em lote, a arquitetura deve permitir trocar arquivo CSV por API, rotina padrao ou programa ABL sem alterar o dominio.

## 7. Mapeamento de dados

Cada fonte externa deve possuir mapper proprio para converter o retorno bruto em modelo interno.

Fluxo recomendado:

```text
Datasul -> DTO fornecedor -> Mapper -> FornecedorDatasul
Receita -> DTO bruto -> Mapper -> CadastroReceita
FornecedorDatasul + CadastroReceita -> Comparador -> ResultadoValidacao
ResultadoValidacao -> Relatorio de inconsistencias -> Aprovacao -> CSV Datasul
```

O ultimo passo pode ser CSV, API, ABL ou outro mecanismo homologado. O restante da aplicacao nao deve conhecer detalhes de campos especificos da API, arquivo ou programa de carga.

## 8. Portas sugeridas

```text
CnpjDataProvider
FornecedorInput
CadastroCnpjProvider
InconsistencyReportWriter
ApprovalInput
UpdateOutput
CacheRepository
ExecutionLogger
Clock
ProgressReporter
SettingsProvider
```

Casos de uso dependem das abstracoes; infraestrutura fornece implementacoes.

## 9. Estrutura sugerida

```text
consulta_cnpj/
|-- src/
|   `-- consulta_cnpj/
|       |-- domain/
|       |   |-- entities/
|       |   |-- value_objects/
|       |   |-- services/
|       |   `-- exceptions/
|       |-- application/
|       |   |-- use_cases/
|       |   |-- ports/
|       |   `-- dto/
|       |-- infrastructure/
|       |   |-- providers/
|       |   |-- datasul/
|       |   |-- http/
|       |   |-- files/
|       |   |-- outputs/
|       |   |-- approvals/
|       |   |-- cache/
|       |   |-- config/
|       |   `-- logging/
|       |-- presentation/
|       |   |-- cli/
|       |   `-- gui/
|       `-- bootstrap.py
|-- tests/
|   |-- unit/
|   |-- integration/
|   |-- regression/
|   `-- fixtures/
|-- docs/
|-- scripts/
|-- pyproject.toml
`-- README.md
```

## 10. Dependencias

Escolhas iniciais recomendadas:

- Python 3.12 ou versao corporativamente homologada;
- `httpx` ou `requests` para HTTP;
- `pydantic` ou dataclasses para DTOs, se fizer sentido;
- `pytest` para testes;
- `ruff` para lint e formatacao;
- `pandas`/`openpyxl` se XLSX for requisito;
- `argparse` ou Typer para CLI;
- Tkinter ou CustomTkinter para GUI, se houver GUI;
- PyInstaller para distribuicao Windows, se necessario.

A selecao definitiva deve ocorrer no inicio da implementacao e ser registrada como ADR quando relevante.

## 11. Configuracao

Configuracoes tecnicas podem incluir:

- tipo de entrada de fornecedores: API, CSV, XLSX ou mock;
- fonte de consulta;
- tipo de saida de inconsistencias: CSV, XLSX, JSON ou API;
- tipo de entrada de aprovacao: CSV, XLSX, API ou interface;
- tipo de saida de atualizacao: CSV, API, ABL ou dry-run;
- forma de extracao Datasul;
- timeout;
- limite de tentativas;
- intervalo entre requisicoes;
- caminho de saida;
- modo debug;
- token ou credencial por variavel de ambiente;
- periodo de referencia da execucao.

Regras essenciais de negocio nao devem ficar em configuracoes arbitrariamente editaveis pelo usuario.

## 12. Excecoes

Separar:

- excecoes de dominio;
- excecoes de aplicacao;
- excecoes de infraestrutura.

Exemplos:

- `CnpjInvalido`;
- `FonteConsultaIndisponivel`;
- `LimiteRequisicoesExcedido`;
- `RespostaFonteInvalida`;
- `FalhaExtracaoDatasul`;
- `LayoutAtualizacaoDatasulIndefinido`;
- `AdaptadorNaoConfigurado`;
- `ContratoAdaptadorViolado`;
- `ArquivoEntradaInvalido`;
- `SemPermissaoGravacao`.

A camada de apresentacao traduz excecoes para mensagens apropriadas.

## 13. Concorrencia

O MVP pode iniciar com processamento sequencial com capacidade de retomada, considerando volume aproximado de 30 mil fornecedores.

Antes de introduzir paralelismo, medir:

- limites da fonte externa;
- estabilidade da API;
- tempo medio de resposta;
- risco de bloqueio;
- consumo de memoria em lotes;
- necessidade de retomar execucao interrompida.

Respeitar termos de uso e limites das fontes consultadas e mais importante que otimizar prematuramente.

## 14. Evolucoes previstas

A arquitetura deve permitir futuramente:

- multiplas fontes;
- fallback entre fontes;
- cache configuravel;
- GUI;
- API local;
- integracao com ERP;
- atualizacao via API Datasul;
- geracao para programa ABL;
- agendamento de consultas;
- historico em banco de dados;
- monitoramento de alteracoes cadastrais.

Essas possibilidades nao devem ampliar o escopo do MVP sem decisao explicita.
