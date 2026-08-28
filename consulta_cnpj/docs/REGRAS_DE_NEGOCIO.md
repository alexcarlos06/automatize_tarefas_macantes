# Regras de Negocio - Consulta CNPJ

## RN-001 - Unidade de processamento

Um fornecedor Datasul com CNPJ e a unidade logica de processamento.

Em lote, cada fornecedor deve gerar exatamente um resultado de validacao ou uma pendencia correspondente.

## RN-002 - Normalizacao

Antes de qualquer validacao ou consulta, o CNPJ deve ser normalizado para conter apenas digitos.

Exemplo:

```text
12.345.678/0001-90
```

torna-se:

```text
12345678000190
```

## RN-003 - Validacao de formato

Um CNPJ valido para consulta deve possuir 14 digitos.

Entradas vazias, com letras ou com quantidade incorreta de digitos devem ser classificadas como invalidas.

## RN-004 - Digitos verificadores

A aplicacao deve validar os digitos verificadores do CNPJ antes de consultar fonte externa.

CNPJs com digitos verificadores invalidos nao devem consumir chamada externa.

## RN-005 - Sequencias invalidas

CNPJs formados por uma unica sequencia repetida devem ser rejeitados.

Exemplos:

```text
00000000000000
11111111111111
99999999999999
```

## RN-006 - Consulta externa

A consulta externa a Receita Federal ou fonte homologada so deve ocorrer quando o CNPJ passar pelas validacoes locais.

Falha na fonte externa deve retornar status controlado, nao excecao bruta para o usuario final.

## RN-007 - Extracao Datasul

A base de fornecedores deve ser obtida por um componente de entrada substituivel.

Implementacoes aceitas:

- API Datasul;
- CSV exportado do Datasul;
- XLSX exportado do Datasul;
- mock para testes.

Enquanto a integracao nao estiver definida, arquivo CSV ou XLSX exportado do ERP pode ser usado como entrada intermediaria.

Cada registro deve preservar o identificador do fornecedor no Datasul para permitir rastreabilidade e eventual atualizacao.

## RN-008 - Modelo cadastral interno

Respostas de APIs externas devem ser convertidas para um modelo interno padronizado.

O dominio nao deve depender diretamente do JSON, XML ou nomenclatura especifica da fonte externa.

## RN-009 - Campos minimos

Quando disponiveis, os seguintes campos devem ser preservados:

- codigo do fornecedor no Datasul;
- CNPJ;
- razao social no Datasul;
- situacao no Datasul;
- razao social na Receita;
- nome fantasia na Receita;
- situacao cadastral na Receita;
- motivo da situacao cadastral;
- data de abertura;
- municipio;
- UF;
- CEP;
- fonte da consulta;
- data/hora da consulta.

## RN-010 - Situacao irregular

Fornecedor deve ser tratado como inconsistente quando a situacao do CNPJ na Receita Federal for diferente de ativo.

Situacoes irregulares incluem, no minimo:

- baixado;
- inapto;
- suspenso;
- nulo;
- qualquer outra situacao diferente de ativo.

## RN-011 - Comparacao Receita x Datasul

A comparacao principal do MVP deve priorizar a situacao cadastral.

Comparacoes adicionais, como razao social, endereco ou UF, podem ser adicionadas apenas quando forem necessarias para a decisao da area responsavel.

## RN-012 - Status da validacao

Cada item deve receber um status padronizado.

Status iniciais sugeridos:

```text
sucesso
ativo_sem_inconsistencia
irregular_receita
cnpj_invalido
sem_cnpj
nao_encontrado
fonte_indisponivel
limite_requisicoes
resposta_incompleta
erro
```

## RN-013 - Relatorio de inconsistencias

O relatorio de inconsistencias deve conter todos os fornecedores que exigem analise da area responsavel.

Devem entrar no relatorio:

- CNPJ diferente de ativo na Receita;
- CNPJ invalido;
- fornecedor sem CNPJ;
- CNPJ nao encontrado;
- consulta nao concluida por falha externa;
- divergencia relevante definida pela area responsavel.

## RN-014 - Aprovacao da area responsavel

Nenhuma atualizacao no Datasul deve ser executada automaticamente apenas pela consulta.

A area responsavel deve revisar o relatorio e marcar quais fornecedores estao aprovados para atualizacao.

## RN-015 - Saida de atualizacao

A saida de atualizacao deve conter somente registros aprovados pela area responsavel.

O layout deve respeitar o mecanismo de carga escolhido para o Datasul.

Implementacoes possiveis:

- CSV de carga;
- chamada de API;
- arquivo para programa ABL;
- dry-run de homologacao.

Campos minimos sugeridos enquanto o layout final nao estiver definido:

```text
codigo_fornecedor
cnpj
situacao_receita
acao_aprovada
observacao
aprovado_em
```

## RN-016 - Pendencias

Um item deve ser tratado como pendencia quando:

- CNPJ for invalido;
- fornecedor nao possuir CNPJ;
- fonte nao encontrar cadastro;
- consulta externa falhar;
- resposta vier incompleta para campos obrigatorios do fluxo;
- ocorrer erro inesperado no item.

Pendencias devem conter mensagem suficiente para suporte e correcao.

## RN-017 - Processamento parcial

Em lote, erro em um CNPJ nao deve interromper os demais.

Itens ja consultados com sucesso nao devem ser descartados por falha posterior.

## RN-018 - Periodicidade

O processo deve ser planejado para execucao semestral.

A execucao deve registrar periodo, data/hora, fonte usada e quantidade de fornecedores avaliados.

## RN-019 - Reprocessamento

A aplicacao deve permitir reprocessar CNPJs pendentes sem depender do lote original inteiro, quando essa funcionalidade for implementada.

## RN-020 - Cache

Cache e opcional no MVP.

Se implementado, deve possuir politica clara de:

- validade;
- origem dos dados;
- limpeza;
- desativacao;
- tratamento de dados pessoais ou sensiveis.

## RN-021 - Limite de requisicoes

A aplicacao deve tratar limites da fonte externa com status proprio.

Para volume aproximado de 30 mil fornecedores, a implementacao deve considerar atraso entre chamadas, retry controlado, pausa, retomada e limites contratuais da fonte utilizada.

## RN-022 - Retry

Retry so deve ser usado para falhas transientes.

Nao repetir automaticamente casos como CNPJ invalido ou nao encontrado.

## RN-023 - Logs

Logs devem registrar:

- inicio e fim da execucao;
- quantidade total;
- quantidade com sucesso;
- quantidade ativa sem inconsistencia;
- quantidade irregular na Receita;
- quantidade aprovada para atualizacao;
- quantidade com pendencia;
- falhas estruturais;
- identificador do item quando necessario.

Evitar gravar respostas completas de APIs externas quando nao forem necessarias para suporte.

## RN-024 - Segredos

Tokens, chaves e credenciais devem vir de variaveis de ambiente, arquivo local nao versionado ou cofre corporativo.

Nunca embutir segredo no codigo-fonte, fixtures ou documentacao.

## RN-025 - Arquivos de saida

Arquivos existentes nao devem ser sobrescritos silenciosamente.

Quando houver colisao, a aplicacao deve:

- gerar nome unico; ou
- pedir confirmacao na interface; ou
- falhar de forma controlada, conforme interface usada.

## RN-026 - Atualizacao em lote no Datasul

A atualizacao em lote no Datasul deve usar mecanismo seguro e homologado.

Ordem de preferencia:

1. rotina padrao/API oficial do Datasul;
2. integracao homologada pela TI;
3. programa ABL especifico que respeite as regras de negocio do ERP.

O mecanismo definitivo ainda esta em aberto.

## RN-027 - Substituicao de componentes

Cada componente externo deve poder ser substituido por outra implementacao do mesmo contrato.

Componentes substituiveis:

- entrada de fornecedores;
- fonte de dados cadastrais;
- saida do relatorio de inconsistencias;
- entrada de aprovacoes;
- saida de atualizacao;
- cache;
- logging tecnico.

Trocar CSV por API, API por arquivo ou CSV por ABL nao deve alterar as regras de validacao, comparacao e aprovacao.

## RN-028 - Dados reais em testes

Testes automatizados devem usar CNPJs ficticios, publicos de exemplo ou dados anonimizados.

Dados reais de clientes, fornecedores ou parceiros nao devem ser versionados.

## RN-029 - Evolucao de fontes

Novas fontes de consulta devem ser adicionadas como adaptadores independentes.

A inclusao de nova fonte deve exigir preferencialmente:

1. novo cliente/adaptador;
2. novo mapper para modelo interno;
3. testes de contrato ou fixtures anonimizadas;
4. registro da decisao quando houver impacto relevante.
