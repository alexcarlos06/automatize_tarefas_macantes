# Regras de Negócio — Separador de Comprovantes Itaú

## RN-001 — Unidade de processamento

Um comprovante é a unidade lógica do processamento.

Uma página física do PDF pode conter:
- um comprovante;
- mais de um comprovante.

Portanto, página não deve ser usada como chave de separação.

## RN-002 — Identificação do início

A aplicação deve reconhecer os layouts homologados por seus marcadores de início.

Padrões iniciais conhecidos:
- `Comprovante de Transferência`;
- `Comprovante de Operação - TED C`;
- `Comprovante de Operação - Transferência de Conta Corrente para Conta Corrente`;
- `Comprovante de Operação - Títulos Outros Bancos`;
- `Comprovante de Operação - Títulos Itaú`.

Novos layouts devem ser adicionados por meio de novos extratores/estratégias, evitando alteração desnecessária das regras existentes.

## RN-003 — Delimitação

O texto `Cortar aqui` é um marcador conhecido de separação.

A estratégia de separação deve combinar:
- identificação de início de comprovante;
- delimitadores conhecidos;
- coordenadas/posicionamento do conteúdo no PDF quando necessário para preservar corretamente o recorte visual.

Não se deve assumir que `Cortar aqui` será a única regra possível para todas as evoluções do produto.

## RN-004 — Dados mínimos

Para que um comprovante seja considerado apto à geração normal, devem ser obtidos:
- data do pagamento;
- favorecido/recebedor;
- valor.

Caso algum campo obrigatório não seja identificado, o item deve ser classificado como pendência.

## RN-005 — Data

A data utilizada para:
- nomenclatura;
- estrutura de diretórios;
- identificador diário

deve ser a data efetiva do pagamento/transferência extraída do comprovante.

Campos conhecidos incluem, conforme layout:
- `data da transferência`;
- `Transferência realizada em`;
- `Pagamento efetuado em`.

`Data de vencimento` de título não substitui a data efetiva do pagamento.

## RN-006 — Favorecido

O favorecido deve representar o recebedor do recurso.

Campos conhecidos:
- `nome do recebedor`;
- `Nome do Favorecido`;
- `Nome do favorecido`;
- `Nome` dentro do bloco de dados da conta creditada.

O extrator deve conhecer o contexto do layout para não confundir o nome do pagador com o favorecido.

## RN-007 — Valor

O valor deve representar o valor efetivamente transferido/pago.

Campos conhecidos:
- `valor`;
- `Valor`;
- `Valor pago`.

O valor deve ser normalizado para nomenclatura mantendo vírgula como separador de centavos.

Exemplo:

```text
R$ 1.113.844,32
```

torna-se:

```text
1113844,32
```

## RN-008 — Identificador diário

Cada comprovante gerado recebe um ID sequencial dentro da data de pagamento.

Formato inicial:

```text
001
002
003
...
```

O sequenciamento deve evitar sobrescrita de arquivo.

Se já existirem comprovantes no diretório diário, a aplicação deve determinar com segurança o próximo identificador disponível em vez de reiniciar cegamente em `001`.

## RN-009 — Nome do arquivo

Formato obrigatório:

```text
id_aammdd_favorecido_valor.pdf
```

Exemplo:

```text
001_120826_CICLO_LOGISTICA_LTDA_1113844,32.pdf
```

## RN-010 — Normalização do favorecido

No nome do arquivo:
- remover acentos;
- remover caracteres especiais;
- não utilizar espaços;
- substituir separação entre palavras por `_`;
- utilizar somente caracteres seguros para Windows;
- manter a extensão `.pdf`.

Exemplo:

```text
COMPANHIA DE GÁS SÃO PAULO
```

torna-se:

```text
COMPANHIA_DE_GAS_SAO_PAULO
```

## RN-011 — Estrutura de diretórios

Formato:

```text
AAAA/MM/DD
```

Exemplo para pagamento em 12/08/2026:

```text
2026/08/12
```

Diretórios inexistentes devem ser criados automaticamente.

## RN-012 — Pendências

Erro individual não deve interromper o lote.

Um comprovante deve entrar na lista de pendências quando, entre outros casos:
- layout não reconhecido;
- data não identificada;
- favorecido não identificado;
- valor não identificado;
- recorte do comprovante não puder ser produzido com segurança;
- geração do PDF individual falhar.

A pendência deve conter informações suficientes para análise posterior.

## RN-013 — Erros críticos

Erros que impedem a execução geral devem interromper o processamento de forma controlada.

Exemplos:
- arquivo de entrada inexistente;
- arquivo inválido ou ilegível;
- diretório de saída inacessível;
- ausência de permissão para gravação;
- impossibilidade de criar a estrutura mínima necessária.

## RN-014 — Permissão

A aplicação não administra permissões de rede.

Caso o usuário não possua permissão:
- capturar a exceção;
- apresentar mensagem clara na interface ou CMD;
- encerrar de forma controlada quando a gravação for indispensável.

## RN-015 — Histórico

Deve existir:

```text
<saida>/Historico
```

O PDF original deve ser copiado para o histórico e renomeado de forma padronizada.

Padrão inicial sugerido:

```text
ORIGINAL_aaaammdd_HHMMSS.pdf
```

A data/hora representa a execução e evita colisões entre processamentos.

O arquivo original não deve ser apagado automaticamente da localização selecionada pelo usuário.

## RN-016 — Histórico do processamento

Cada execução deve registrar um resumo contendo, no mínimo:
- data/hora;
- nome do arquivo original;
- quantidade de comprovantes identificados;
- quantidade gerada;
- quantidade pendente;
- status final.

Não é necessário registrar o usuário para fins de auditoria.

## RN-017 — Debug

Debug é uma opção explícita do usuário.

Quando ativo:
- criar `<saida>/Debug`;
- gerar log técnico incremental;
- incluir informações de diagnóstico.

Quando inativo:
- não gerar arquivo técnico detalhado;
- continuar apresentando informações operacionais na GUI/CMD.

## RN-018 — Integridade

Um arquivo existente nunca deve ser sobrescrito silenciosamente.

Qualquer colisão deve ser evitada pelo ID ou tratada como exceção controlada.

## RN-019 — Processamento parcial

Comprovantes já gerados com sucesso não devem ser removidos apenas porque outro comprovante do mesmo lote falhou.

## RN-020 — Evolução de layouts

Layouts bancários devem ser tratados como estratégias independentes.

A inclusão de um novo layout deve exigir preferencialmente:
1. novo extrator;
2. testes do novo layout;
3. registro do layout suportado;

sem alterar o domínio central.
