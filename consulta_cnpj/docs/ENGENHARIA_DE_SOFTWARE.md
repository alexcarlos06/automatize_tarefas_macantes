# Engenharia de Software - Consulta CNPJ

## 1. Objetivo

Definir praticas para que a solucao possa ser mantida, testada e distribuida com seguranca.

## 2. Repositorio

Codigo-fonte deve permanecer em repositorio controlado pela TI.

Estrutura minima sugerida:

```text
/
|-- src/
|-- tests/
|-- docs/
|-- scripts/
|-- .gitignore
|-- pyproject.toml
`-- README.md
```

Arquivos com dados reais de clientes, fornecedores ou parceiros nao devem ser versionados.

## 3. Dados de teste

Fixtures permanentes devem:

- usar CNPJs ficticios, publicos de exemplo ou anonimizados;
- simular fornecedores Datasul com codigo interno;
- cobrir CNPJ valido e invalido;
- cobrir respostas de sucesso;
- cobrir situacao ativa, baixada, inapta e suspensa;
- cobrir fonte indisponivel;
- cobrir limite de requisicoes;
- cobrir resposta incompleta.

## 4. Testes

Framework recomendado: `pytest`.

### 4.1 Unitarios

Cobrir:

- remocao de mascara;
- validacao de tamanho;
- rejeicao de sequencias repetidas;
- calculo dos digitos verificadores;
- formatacao de CNPJ;
- status de consulta;
- mapeamento de resposta externa;
- comparacao Receita x Datasul;
- classificacao de inconsistencias;
- geracao de saida de atualizacao aprovada;
- normalizacao de campos de saida.

### 4.2 Aplicacao

Cobrir:

- extracao ou leitura de fornecedores Datasul;
- CNPJ invalido sem chamada externa;
- fonte externa retornando nao encontrado;
- fornecedor com CNPJ ativo;
- fornecedor com CNPJ irregular;
- timeout;
- limite de requisicoes;
- resposta incompleta;
- lote com sucesso parcial.

### 4.3 Integracao

Cobrir:

- leitura de arquivo de entrada;
- exportacao de relatorio de inconsistencias;
- exportacao de saida de atualizacao;
- configuracao de fonte;
- mock de API Datasul;
- contratos dos adaptadores substituiveis;
- logs operacionais;
- diretorio de saida;
- mock ou sandbox de fonte externa.

### 4.4 Regressao

Cada bug real corrigido deve gerar uma fixture anonimizada e um teste de regressao.

## 5. Cobertura

Nao utilizar percentual de cobertura como unico indicador.

Como meta inicial, buscar cobertura alta das regras de dominio e mapeadores, priorizando caminhos criticos e cenarios de erro.

## 6. Qualidade

Antes de gerar uma release:

- testes aprovados;
- lint aprovado;
- formatacao aprovada;
- type checking, caso adotado;
- teste de consulta individual;
- teste de lote representativo;
- teste de geracao do relatorio de inconsistencias;
- teste de geracao da saida aprovada;
- teste de troca de adaptadores sem alterar o caso de uso;
- teste com falha de rede simulada;
- teste do executavel Windows, se houver empacotamento.

Ferramentas sugeridas:

- Ruff;
- pytest;
- mypy/pyright, se adotado;
- pre-commit, se adotado.

## 7. Versionamento

Usar versionamento semantico quando aplicavel:

```text
MAJOR.MINOR.PATCH
```

Exemplos:

- `0.1.0` - primeiro MVP interno;
- `0.2.0` - integracao Datasul;
- `0.2.1` - correcao sem alteracao funcional relevante;
- `1.0.0` - primeira versao considerada estavel.

## 8. Git

Sugestao:

- `main` protegida;
- desenvolvimento por branches;
- Pull Request;
- revisao por TI;
- commits pequenos e descritivos.

## 9. Build

Se houver distribuicao para usuario final, o build deve produzir artefato identificado.

Exemplo:

```text
ConsultaCnpjFornecedores-0.1.0.exe
```

Preferencias:

- build reproduzivel;
- dependencias fixadas;
- versao incorporada;
- changelog basico.

## 10. CLI e GUI

Nao duplicar logica.

```text
GUI ----\
        > Casos de uso
CLI ----/
```

Testes de dominio e aplicacao nao devem depender da interface.

## 11. Logging

Existem dois niveis:

### Operacional

Sempre disponivel para GUI/CLI:

- inicio;
- progresso;
- totais;
- pendencias;
- inconsistencias por situacao;
- aprovados para atualizacao;
- conclusao.

### Tecnico

Somente quando debug estiver ativo:

- stack traces;
- fonte selecionada;
- modo de extracao Datasul;
- tempos de requisicao;
- retries;
- caminhos relevantes;
- detalhes tecnicos de falhas.

Nao registrar tokens, segredos ou respostas completas desnecessariamente.

## 12. Performance

Criar benchmark com lote representativo para aproximar a execucao real de 30 mil fornecedores.

Medir:

- tempo total;
- tempo por CNPJ;
- tempo de extracao Datasul;
- quantidade de falhas;
- impacto de timeout;
- impacto de retry;
- consumo de memoria.

Metas devem considerar os limites da fonte externa.

## 13. Robustez

Principios:

- nunca sobrescrever arquivos silenciosamente;
- erro individual nao encerra lote;
- erro estrutural encerra de forma controlada;
- mensagens devem orientar suporte;
- chamadas externas devem ter timeout;
- retry deve ser limitado;
- execucao longa deve permitir retomada ou reprocessamento de pendencias;
- configuracoes obrigatorias devem ser validadas no inicio.

## 14. Seguranca

- nenhum segredo embutido;
- arquivos reais fora do Git;
- logs com minimo necessario;
- respeitar LGPD e regras internas;
- validar entrada antes de consultar fonte externa;
- nao enviar dados para fontes nao aprovadas;
- registrar fonte e horario da consulta para rastreabilidade.
- nao atualizar o Datasul sem aprovacao previa da area responsavel.

## 15. ADR - Architecture Decision Records

Decisoes relevantes devem ser registradas em:

```text
docs/adr/
```

Exemplos:

- escolha da fonte de consulta;
- escolha do mecanismo de extracao Datasul;
- escolha do mecanismo de atualizacao em lote Datasul;
- escolha dos contratos de componentes substituiveis;
- escolha da biblioteca HTTP;
- estrategia de cache;
- formato de entrada e saida;
- escolha GUI;
- empacotamento.

Formato:

```text
# ADR-001 - Titulo
Status: Aceito

## Contexto
...

## Decisao
...

## Consequencias
...
```

## 16. Definition of Done

Uma funcionalidade e considerada concluida quando:

- regra implementada;
- testes automatizados;
- documentacao atualizada quando necessario;
- tratamento de erro implementado;
- lint e testes aprovados;
- comportamento validado na interface impactada;
- relatorio de inconsistencias validado;
- saida de atualizacao gerada apenas a partir de aprovacoes;
- adaptadores novos cobertos por testes de contrato;
- nao contem dados reais ou segredos no repositorio.
