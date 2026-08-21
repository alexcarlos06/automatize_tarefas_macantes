# Engenharia de Software — Separador de Comprovantes Itaú

## 1. Objetivo

Definir práticas para que a solução possa ser mantida pelo time de TI e distribuída com segurança ao usuário final.

## 2. Repositório

Código-fonte deve permanecer em repositório controlado pela TI.

Estrutura mínima:

```text
/
├── src/
├── tests/
├── docs/
├── scripts/
├── .gitignore
├── pyproject.toml
└── README.md
```

Arquivos financeiros reais não devem ser versionados.

## 3. Dados de teste

O PDF real utilizado na análise contém dados financeiros e deve ficar fora do Git.

Fixtures permanentes devem:
- utilizar dados fictícios ou anonimizados;
- representar todos os layouts homologados;
- incluir cenários inválidos.

## 4. Testes

Framework recomendado: `pytest`.

### 4.1 Unitários

Cobrir:
- normalização de favorecido;
- normalização de valor;
- data;
- nomenclatura;
- ID diário;
- regras de pendência;
- seleção de extrator.

### 4.2 Extratores

Um conjunto de testes por layout:
- transferência/PIX;
- TED;
- transferência entre contas;
- títulos outros bancos;
- títulos Itaú.

Cada extrator deve testar:
- caso válido;
- ausência de favorecido;
- ausência de valor;
- ausência de data;
- variações de capitalização/espaçamento relevantes.

### 4.3 Integração

Cobrir:
- PDF com um comprovante;
- PDF com múltiplos comprovantes;
- múltiplos comprovantes na mesma página;
- criação de estrutura de saída;
- histórico;
- pendências;
- debug.

### 4.4 Regressão

Cada novo layout ou bug real corrigido deve gerar uma fixture anonimizada e um teste de regressão.

## 5. Cobertura

Não utilizar percentual de cobertura como único indicador.

Como meta inicial, buscar cobertura alta das regras de domínio e dos extratores (por exemplo, >= 85%), priorizando caminhos críticos e cenários de erro.

## 6. Qualidade

Antes de gerar uma release:
- testes aprovados;
- lint aprovado;
- formatação aprovada;
- type checking, caso adotado;
- teste do executável Windows;
- teste GUI;
- teste CLI;
- teste em diretório de rede homologado.

Ferramentas sugeridas:
- Ruff;
- Black (ou formatter do Ruff);
- mypy/pyright, se adotado;
- pytest.

## 7. Versionamento

Usar versionamento semântico quando aplicável:

```text
MAJOR.MINOR.PATCH
```

Exemplos:
- `0.1.0` — primeiro MVP interno;
- `0.2.0` — novo layout;
- `0.2.1` — correção sem alteração funcional relevante;
- `1.0.0` — primeira versão considerada estável.

## 8. Git

Sugestão:
- `main` protegida;
- desenvolvimento por branches;
- Pull Request;
- revisão por TI;
- commits pequenos e descritivos.

## 9. Build

O build deve produzir executável Windows.

Preferência:
- build reproduzível;
- dependências fixadas;
- versão incorporada;
- artefato identificado.

Exemplo:

```text
SeparadorComprovantesItau-0.1.0.exe
```

## 10. CLI e GUI

Não duplicar lógica.

```text
GUI ----\
        > ProcessarArquivo
CLI ----/
```

Testes de domínio/aplicação não devem depender da interface.

## 11. Logging

Existem dois níveis:

### Operacional
Sempre disponível para GUI/CLI:
- início;
- progresso;
- totais;
- pendências;
- conclusão.

### Técnico
Somente quando debug estiver ativo:
- stack traces;
- extrator selecionado;
- etapas de leitura/recorte;
- caminhos relevantes;
- tempos;
- detalhes técnicos de falhas.

Não registrar conteúdo financeiro integral desnecessariamente no log.

## 12. Performance

Criar benchmark com fixture representativa.

Medir:
- tempo total;
- tempo por comprovante;
- memória;
- desempenho em disco local;
- desempenho em pasta de rede.

Meta de performance deve ser definida após baseline do primeiro protótipo, evitando estimativa arbitrária.

## 13. Robustez

Princípios:
- nunca sobrescrever silenciosamente;
- operações de arquivo devem validar resultado;
- histórico deve ser preservado;
- erro individual não encerra lote;
- erro estrutural encerra de forma controlada;
- mensagens devem orientar suporte.

## 14. Segurança

- processamento local;
- nenhuma dependência de IA/API externa no MVP;
- nenhum segredo embutido;
- arquivos financeiros fora do Git;
- logs com mínimo necessário;
- respeitar permissões do Windows/rede.

## 15. ADR — Architecture Decision Records

Decisões relevantes devem ser registradas em:

```text
docs/adr/
```

Exemplos:
- escolha da biblioteca PDF;
- escolha GUI;
- escolha CLI;
- formato do histórico;
- estratégia de recorte;
- empacotamento.

Formato:

```text
# ADR-001 — Título
Status: Aceito

## Contexto
...

## Decisão
...

## Consequências
...
```

## 16. Definition of Done

Uma funcionalidade é considerada concluída quando:
- regra implementada;
- testes automatizados;
- documentação atualizada quando necessário;
- tratamento de erro implementado;
- lint/testes aprovados;
- comportamento validado em GUI e/ou CLI conforme impacto;
- não contém dados financeiros reais no repositório.
