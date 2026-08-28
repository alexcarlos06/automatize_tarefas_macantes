# Documentacao para agentes - Consulta CNPJ

Esta pasta contem as instrucoes que devem orientar agentes e desenvolvedores no projeto `consulta_cnpj`.

Leia os arquivos nesta ordem antes de implementar qualquer funcionalidade:

1. `ESPECIFICACAO.md`
2. `REGRAS_DE_NEGOCIO.md`
3. `ARQUITETURA.md`
4. `COMPONENTES_SUBSTITUIVEIS.md`
5. `ENGENHARIA_DE_SOFTWARE.md`

## Escopo confirmado

O projeto tem como objetivo automatizar a validacao cadastral semestral de aproximadamente 30 mil fornecedores do TOTVS Datasul em relacao a situacao cadastral do CNPJ na Receita Federal.

A automacao deve:

- extrair fornecedores do Datasul, preferencialmente por API;
- normalizar e validar os CNPJs;
- obter dados oficiais da Receita Federal ou fonte oficial homologada;
- comparar situacao cadastral Receita x Datasul;
- gerar relatorio de inconsistencias para analise da area responsavel;
- gerar saida de atualizacao somente apos aprovacao das inconsistencias;
- preparar a atualizacao em lote no Datasul por mecanismo seguro, ainda a definir.

## Diretriz de componentes substituiveis

Nenhuma parte do fluxo deve ficar presa a uma implementacao especifica.

Entrada, consulta Receita, aprovacao, saida e atualizacao devem ser tratados como componentes substituiveis por contrato. Exemplos:

- entrada de fornecedores por API Datasul, CSV ou XLSX;
- consulta cadastral por API oficial, fonte homologada ou mock de testes;
- relatorio de inconsistencias em CSV, XLSX ou API;
- saida de atualizacao por CSV, API Datasul ou programa ABL.

Detalhes estao em `COMPONENTES_SUBSTITUIVEIS.md`.

## Pontos em aberto

- qual campo/informacao do Datasul sera alterado quando o fornecedor estiver irregular;
- qual mecanismo sera usado para atualizacao em lote: rotina padrao, API ou programa ABL especifico;
- qual fonte oficial sera usada para os dados da Receita Federal e seus limites para 30 mil consultas.
