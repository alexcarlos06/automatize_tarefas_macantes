# Componentes Substituiveis - Consulta CNPJ

## 1. Objetivo

Definir os pontos de extensao do projeto para que cada parte do fluxo possa ser trocada sem alterar as regras de negocio centrais.

A implementacao deve depender de contratos na camada de aplicacao. APIs, arquivos, banco, CSV, XLSX, ABL e outras tecnologias devem ficar em adaptadores de infraestrutura.

## 2. Principio central

O dominio conhece apenas conceitos de negocio:

- fornecedor;
- CNPJ;
- cadastro Receita;
- validacao;
- inconsistencia;
- aprovacao;
- atualizacao.

O dominio nao deve saber se os dados vieram de API, arquivo, banco, tela, planilha ou rotina ABL.

## 3. Fluxo por componentes

```text
FornecedorInput
    -> CnpjValidator
    -> CadastroCnpjProvider
    -> CadastroComparator
    -> InconsistencyOutput
    -> ApprovalInput
    -> UpdateOutput
```

Cada bloco deve ser substituivel por outra implementacao do mesmo contrato.

## 4. Entrada de fornecedores

Contrato conceitual:

```python
class FornecedorInput:
    def listar(self) -> Iterable[FornecedorBruto]:
        ...
```

Implementacoes possiveis:

- `DatasulApiFornecedorInput`;
- `CsvFornecedorInput`;
- `XlsxFornecedorInput`;
- `MockFornecedorInput`;
- `DatabaseFornecedorInput`, se futuramente houver base intermediaria.

Regras:

- todos os adaptadores devem entregar o mesmo modelo interno;
- validacao de layout de arquivo pertence ao adaptador de arquivo;
- autenticacao e paginacao da API pertencem ao adaptador de API;
- o caso de uso nao deve conter `if api`, `if csv` ou `if xlsx`.

## 5. Fonte cadastral de CNPJ

Contrato conceitual:

```python
class CadastroCnpjProvider:
    def consultar(self, cnpj: Cnpj) -> ResultadoCadastroCnpj:
        ...
```

Implementacoes possiveis:

- `ReceitaFederalProvider`;
- `FonteHomologadaProvider`;
- `BrasilApiProvider`, se aprovada;
- `SerproProvider`, se contratado;
- `CachedCadastroCnpjProvider`;
- `MockCadastroCnpjProvider`.

Regras:

- o provider deve retornar status padronizado;
- limites de requisicao e retry pertencem ao provider ou a decorator tecnico;
- resposta bruta da API nao deve sair da infraestrutura;
- qualquer fonte nova exige testes de mapper e tratamento de erro.

## 6. Saida do relatorio de inconsistencias

Contrato conceitual:

```python
class InconsistencyOutput:
    def escrever(self, resultado: ResultadoValidacaoLote) -> SaidaGerada:
        ...
```

Implementacoes possiveis:

- `CsvInconsistencyOutput`;
- `XlsxInconsistencyOutput`;
- `ApiInconsistencyOutput`;
- `JsonInconsistencyOutput`.

Regras:

- o conteudo logico do relatorio deve ser o mesmo em qualquer saida;
- mudanca de formato nao deve alterar a comparacao Receita x Datasul;
- saidas em arquivo nao devem sobrescrever silenciosamente;
- saidas em API devem tratar erro por lote e por item quando aplicavel.

## 7. Entrada de aprovacao

Contrato conceitual:

```python
class ApprovalInput:
    def carregar_aprovacoes(self) -> Iterable[AprovacaoAtualizacao]:
        ...
```

Implementacoes possiveis:

- `CsvApprovalInput`;
- `XlsxApprovalInput`;
- `ApiApprovalInput`;
- `ManualApprovalInput`, se houver interface futura.

Regras:

- somente itens aprovados podem seguir para atualizacao;
- aprovacao deve referenciar o fornecedor de forma rastreavel;
- item nao aprovado nao deve aparecer na saida de atualizacao;
- aprovacao invalida deve gerar pendencia, nao atualizacao.

## 8. Saida de atualizacao

Contrato conceitual:

```python
class UpdateOutput:
    def enviar(self, atualizacoes: Iterable[AtualizacaoFornecedor]) -> ResultadoAtualizacao:
        ...
```

Implementacoes possiveis:

- `CsvDatasulUpdateOutput`;
- `DatasulApiUpdateOutput`;
- `AblBatchUpdateOutput`;
- `DryRunUpdateOutput`.

Regras:

- a regra de quem pode ser atualizado fica no dominio/aplicacao;
- formato fisico de carga fica no adaptador;
- atualizacao direta em API deve ter modo de simulacao quando possivel;
- programa ABL deve receber dados ja validados e aprovados;
- todo mecanismo deve gerar rastreabilidade do que foi enviado.

## 9. Configuracao de componentes

A selecao de adaptadores deve ser feita por configuracao ou bootstrap.

Exemplo conceitual:

```text
input.type=csv
input.path=fornecedores_datasul.csv
cnpj_provider.type=receita_federal
inconsistency_output.type=xlsx
approval_input.type=csv
update_output.type=csv_datasul
```

O dominio e os casos de uso nao devem ler diretamente variaveis de ambiente, argumentos CLI ou arquivos de configuracao.

## 10. Testes obrigatorios por componente

Cada adaptador deve ter testes para:

- caso de sucesso;
- entrada ou resposta invalida;
- campos obrigatorios ausentes;
- erro externo controlado;
- mapeamento para o modelo interno;
- preservacao de identificadores de rastreabilidade.

Tambem deve existir teste de contrato para garantir que implementacoes diferentes entregam o mesmo comportamento esperado ao caso de uso.

## 11. Regra para agentes

Ao implementar uma nova parte do fluxo:

1. criar ou reutilizar uma porta na camada de aplicacao;
2. implementar o adaptador em infraestrutura;
3. mapear dados externos para modelo interno;
4. adicionar testes unitarios e de contrato;
5. registrar ADR quando a escolha do componente tiver impacto relevante;
6. evitar espalhar detalhes de API, CSV, XLSX ou ABL fora do adaptador.

