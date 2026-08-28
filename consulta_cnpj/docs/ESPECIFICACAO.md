# Especificacao do Projeto - Consulta CNPJ

## 1. Nome do projeto

**Consulta CNPJ**

Nome tecnico sugerido: `consulta_cnpj`.

## 2. Objetivo

Desenvolver uma automacao em Python para realizar, semestralmente, a validacao cadastral de aproximadamente 30 mil fornecedores do TOTVS Datasul em relacao a situacao cadastral do CNPJ na Receita Federal.

A solucao deve identificar principalmente fornecedores cujo CNPJ esteja baixado, inapto, suspenso ou em qualquer situacao diferente de ativo, gerando um relatorio de inconsistencias para analise antes de qualquer alteracao no ERP.

Apos validacao das inconsistencias pela area responsavel, o Python deve gerar uma saida de atualizacao somente com os fornecedores aprovados. Essa saida pode ser CSV, chamada de API, arquivo para programa ABL ou outro mecanismo homologado.

## 3. Contexto do processo

O fluxo esperado envolve:

1. extrair fornecedores do Datasul por API ou rotina homologada;
2. normalizar e validar os CNPJs;
3. consultar dados oficiais da Receita Federal ou fonte oficial homologada;
4. comparar Receita x Datasul;
5. gerar relatorio de inconsistencias;
6. submeter inconsistencias para validacao da area responsavel;
7. gerar saida de atualizacao apenas para fornecedores aprovados;
8. atualizar o Datasul em lote por mecanismo seguro, ainda a definir.

## 4. Escopo do MVP

O MVP deve:

- extrair ou receber base de fornecedores do Datasul por componente substituivel;
- processar aproximadamente 30 mil fornecedores em uma execucao semestral;
- validar formato e digitos verificadores do CNPJ;
- consultar situacao cadastral em fonte oficial da Receita Federal ou fonte homologada;
- identificar fornecedores com situacao diferente de ativo;
- comparar campos relevantes entre Receita e Datasul;
- tratar CNPJ invalido sem acionar consulta externa;
- tratar falha de rede, limite de requisicoes e resposta incompleta;
- gerar relatorio de inconsistencias para analise;
- gerar saida de atualizacao somente com itens aprovados pela area responsavel;
- registrar rastreabilidade da execucao;
- possuir logs operacionais;
- manter segredo, token ou chave fora do codigo-fonte;
- possuir testes automatizados para validacao e normalizacao.

## 5. Entrada

### 5.1 Diretriz

A entrada de fornecedores deve ser um componente substituivel. A aplicacao deve aceitar implementacoes por API Datasul, arquivo CSV, arquivo XLSX ou mock de testes, desde que todas entreguem o mesmo modelo interno.

O caso de uso de validacao nao deve depender do mecanismo fisico de entrada.

### 5.2 Base Datasul por API

A entrada preferencial deve ser a base de fornecedores extraida do TOTVS Datasul por API ou rotina homologada.

Campos minimos esperados da base Datasul:

```text
codigo_fornecedor
cnpj
razao_social_datasul
situacao_datasul
```

Campos adicionais podem ser incluidos conforme disponibilidade:

```text
nome_fantasia_datasul
endereco_datasul
municipio_datasul
uf_datasul
inscricao_estadual
grupo_fornecedor
data_ultima_atualizacao
```

### 5.3 Arquivo intermediario

Enquanto a integracao com API Datasul nao estiver definida, a aplicacao pode aceitar arquivo CSV ou XLSX exportado do ERP.

A entrada deve conter uma coluna de CNPJ claramente identificada.

```text
cnpj
CNPJ
documento
```

Linhas invalidas devem ser registradas no resultado como pendencia, sem interromper o arquivo inteiro.

## 6. Saida

### 6.1 Diretriz

As saidas tambem devem ser componentes substituiveis.

O conteudo logico deve permanecer o mesmo, independentemente do destino ser arquivo CSV, XLSX, JSON, API ou outro adaptador.

### 6.2 Relatorio de inconsistencias

Resultado minimo sugerido:

```text
codigo_fornecedor
cnpj
cnpj_formatado
razao_social_datasul
situacao_datasul
razao_social_receita
situacao_receita
data_situacao_receita
motivo_situacao_receita
tipo_inconsistencia
acao_sugerida
aprovado_para_atualizacao
observacao_area_responsavel
fonte
status_consulta
mensagem
consultado_em
```

### 6.3 Saida de atualizacao

A saida de atualizacao deve conter somente fornecedores aprovados pela area responsavel.

Implementacoes possiveis:

- CSV para carga no Datasul;
- API Datasul;
- arquivo de entrada para programa ABL;
- dry-run para homologacao.

O layout definitivo depende da informacao que sera alterada no Datasul e do mecanismo de carga escolhido.

## 7. Interfaces

O nucleo da aplicacao deve ser independente da interface.

Interfaces possiveis:

- CLI para automacao e suporte tecnico;
- arquivo de configuracao para execucao semestral;
- modulo Python reutilizavel por outros scripts;
- GUI futura, se a area responsavel precisar operar sem linha de comando.

GUI e CLI devem chamar os mesmos casos de uso.

## 8. Linha de comando

Exemplo conceitual:

```text
consulta-cnpj --cnpj "00.000.000/0000-00"
consulta-cnpj validar-fornecedores --input-type csv --input "fornecedores_datasul.csv" --output-type xlsx --output-dir "saida"
consulta-cnpj validar-fornecedores --input-type datasul-api --output-type api
consulta-cnpj gerar-atualizacao --approval-type csv --approvals "saida/inconsistencias_aprovadas.csv" --update-output-type csv --output "saida/atualizacao_datasul.csv"
```

Os parametros definitivos devem ser definidos durante a implementacao e documentados no README do projeto.

## 9. Fontes externas e ERP

A escolha da fonte da Receita Federal e do mecanismo Datasul deve ser registrada em ADR quando envolver trade-offs relevantes, como:

- custo;
- limite de requisicoes;
- necessidade de token;
- disponibilidade;
- cobertura dos dados;
- termos de uso;
- estabilidade da API;
- regras de negocio do ERP;
- forma segura de atualizacao em lote.

Nao acoplar regras de negocio ao formato bruto da API externa.

## 10. Substituicao de componentes

O projeto deve permitir substituicao dos seguintes componentes sem alterar regras de negocio:

```text
Entrada de fornecedores: API Datasul, CSV, XLSX, mock
Fonte Receita/CNPJ: API oficial, fonte homologada, cache, mock
Relatorio de inconsistencias: CSV, XLSX, JSON, API
Entrada de aprovacao: CSV, XLSX, API, interface futura
Saida de atualizacao: CSV, API Datasul, ABL, dry-run
```

Cada substituicao deve implementar a porta correspondente descrita em `COMPONENTES_SUBSTITUIVEIS.md`.

## 11. Tratamento de falhas

Falhas individuais:

- CNPJ invalido;
- CNPJ nao encontrado;
- fornecedor sem CNPJ;
- situacao Receita diferente de ativo;
- resposta incompleta;
- limite de requisicoes;
- falha temporaria de rede.

Essas falhas devem ser registradas no item correspondente e nao devem interromper o lote.

Falhas estruturais:

- arquivo de entrada inexistente;
- arquivo sem coluna de CNPJ;
- diretorio de saida inacessivel;
- configuracao obrigatoria ausente;
- fonte externa indisponivel de forma generalizada;
- falha na extracao Datasul.
- adaptador de entrada ou saida nao configurado.

Essas falhas podem encerrar a execucao de forma controlada.

## 12. Requisitos nao funcionais

### RNF-01 - Plataforma

Priorizar Windows, sem impedir uso em outros sistemas quando viavel.

### RNF-02 - Manutenibilidade

Codigo modular, testavel e documentado para manutencao por TI.

### RNF-03 - Testabilidade

Validacao de CNPJ, normalizacao, mapeamento de respostas e tratamento de erro devem possuir testes automatizados.

### RNF-04 - Seguranca

Tokens, chaves e configuracoes sensiveis nao devem ser versionados.

### RNF-05 - Privacidade

Evitar registrar dados desnecessarios em logs. Respeitar a finalidade da consulta e as regras internas de tratamento de dados.

### RNF-06 - Observabilidade

Registrar progresso, totais, falhas por item, inconsistencias encontradas, aprovados para atualizacao e resumo final.

### RNF-07 - Robustez

Falhas individuais nao devem invalidar itens processados corretamente.

### RNF-08 - Substituibilidade

Entrada, fonte cadastral, relatorio, aprovacao e atualizacao devem ser substituiveis por adaptadores sem alterar dominio ou casos de uso principais.

## 13. Fora do escopo inicial

- automacao de navegador para consulta manual;
- enriquecimento comercial amplo;
- captura de socios sem confirmacao de necessidade;
- armazenamento permanente em banco de dados;
- atualizacao automatica sem validacao humana;
- monitoramento continuo de alteracoes cadastrais;
- uso obrigatorio de IA.

## 14. Criterios de aceite do MVP

O MVP sera considerado funcional quando:

1. validar CNPJs com e sem mascara;
2. rejeitar CNPJs invalidos antes de consultar fonte externa;
3. consultar CNPJ valido em fonte configurada;
4. comparar situacao cadastral Receita x Datasul;
5. identificar fornecedores com situacao diferente de ativo;
6. processar lote sem parar por erro individual;
7. gerar relatorio de inconsistencias;
8. gerar saida de atualizacao somente para itens aprovados;
9. registrar resumo da execucao;
10. tratar erro de rede de forma clara;
11. manter segredos fora do repositorio;
12. passar pela suite de testes definida para o MVP.
