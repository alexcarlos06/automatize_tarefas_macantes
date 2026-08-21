# Especificação do Projeto — Separador de Comprovantes Itaú

## 1. Nome do projeto

**Separador de Comprovantes Itaú**

Diretório da automação no monorepo:

```text
separador_comprovantes_itau/
```

Esta solução faz parte do repositório único `alexcarlos06/automatize_tarefas_macantes`. Não deve ser tratada como um repositório separado.

## 2. Objetivo

Desenvolver uma aplicação Windows em Python capaz de receber um PDF consolidado de comprovantes extraído do Bankline Itaú, identificar os comprovantes existentes, separar cada comprovante em um PDF individual, extrair seus dados principais, padronizar o nome do arquivo e armazená-lo automaticamente em uma estrutura de diretórios baseada na data do pagamento.

O código-fonte será mantido e evoluído pelo time de TI dentro do monorepo. O usuário final receberá um executável, sem necessidade de possuir ambiente Python configurado.

## 3. Contexto do processo atual

Atualmente o time financeiro:
1. acessa o Bankline Itaú;
2. baixa um PDF contendo vários comprovantes;
3. identifica manualmente cada comprovante;
4. separa cada comprovante em um novo PDF;
5. identifica data, favorecido e valor;
6. renomeia o arquivo;
7. grava o arquivo em uma pasta de rede.

O processo é repetitivo e pode atingir aproximadamente 1.000 comprovantes por dia.

## 4. Escopo do MVP

O MVP deve:
- funcionar exclusivamente em Windows;
- possuir interface gráfica;
- permitir execução por linha de comando;
- permitir seleção do PDF de entrada;
- permitir seleção do diretório raiz de saída;
- identificar múltiplos layouts de comprovantes Itaú;
- suportar mais de um comprovante na mesma página;
- extrair data de pagamento, favorecido e valor;
- gerar PDFs individuais;
- organizar os comprovantes em `ano/mês/dia`;
- gerar identificador sequencial diário;
- tratar falhas individualmente sem interromper todo o processamento;
- apresentar log operacional na interface;
- possuir modo debug opcional;
- registrar histórico das execuções;
- preservar o PDF original no histórico com nome padronizado;
- gerar lista de pendências;
- apresentar erros de acesso/permissão ao usuário.

## 5. Entrada

### 5.1 Arquivo

Arquivo PDF consolidado gerado pelo Itaú.

O PDF possui camada textual pesquisável, portanto OCR não faz parte do MVP.

### 5.2 Características observadas

Foram identificados layouts como:
- `Comprovante de Transferência`;
- `Comprovante de Operação - TED C`;
- `Comprovante de Operação - Transferência de Conta Corrente para Conta Corrente`;
- `Comprovante de Operação - Títulos Outros Bancos`;
- `Comprovante de Operação - Títulos Itaú`.

Também foi observado o marcador textual `Cortar aqui` entre diversos comprovantes.

A implementação não deve assumir que uma página corresponde a um único comprovante.

## 6. Saída

### 6.1 Estrutura

```text
<diretorio_saida>/
├── Historico/
├── Debug/                  # somente quando debug estiver habilitado
└── AAAA/
    └── MM/
        └── DD/
            ├── 001_aammdd_favorecido_valor.pdf
            ├── 002_aammdd_favorecido_valor.pdf
            └── ...
```

A pasta diária deve ser determinada pela data efetiva do pagamento identificada no comprovante, e não pela data de execução da aplicação.

### 6.2 Nome do comprovante

Formato:

```text
id_aammdd_favorecido_valor.pdf
```

Exemplo:

```text
001_120826_CICLO_LOGISTICA_LTDA_1113844,32.pdf
```

Regras detalhadas estão em `REGRAS_DE_NEGOCIO.md`.

## 7. Interface gráfica

A interface deve permitir:
- selecionar arquivo PDF de entrada;
- selecionar diretório de saída;
- habilitar/desabilitar debug;
- iniciar processamento;
- acompanhar mensagens de execução;
- visualizar contadores de sucesso e pendência;
- visualizar mensagem final do processamento.

A interface não deve conter regras de negócio. Ela apenas aciona os casos de uso da aplicação.

## 8. Linha de comando

A aplicação deve possuir execução via CMD utilizando o mesmo núcleo da interface gráfica.

Exemplo conceitual:

```text
SeparadorComprovantesItau.exe --input "C:\Entrada\comprovantes.pdf" --output "\\servidor\financeiro" --debug
```

Os parâmetros definitivos serão estabelecidos durante a implementação.

## 9. Debug

Quando debug estiver desabilitado:
- não gerar arquivo detalhado de log;
- apresentar acompanhamento somente na interface/console;
- manter apenas os registros funcionais obrigatórios de histórico e pendência.

Quando habilitado:
- criar a pasta `Debug` na raiz da saída, se necessário;
- gerar arquivo de log incremental por execução;
- registrar etapas técnicas suficientes para diagnóstico pelo suporte de TI;
- não impedir o processamento caso um comprovante individual apresente erro.

## 10. Histórico

A raiz de saída deve possuir `Historico`.

O histórico deve preservar:
- PDF original utilizado no processamento, com nome padronizado;
- resumo do processamento;
- informações necessárias para rastrear resultados e pendências.

Não existe requisito de auditoria formal por usuário.

## 11. Tratamento de falhas

Falha em um comprovante:
- registrar a ocorrência;
- adicionar à lista de pendências;
- continuar os demais comprovantes.

Falhas que inviabilizam toda a execução, como impossibilidade de ler a entrada ou gravar no destino:
- interromper de forma controlada;
- apresentar mensagem clara na GUI ou CMD;
- não ocultar a causa técnica quando debug estiver ativo.

Falha de permissão no diretório de rede deve ser tratada explicitamente.

## 12. Requisitos não funcionais

### RNF-01 — Plataforma
Windows.

### RNF-02 — Distribuição
Executável independente para usuário final.

### RNF-03 — Manutenibilidade
Código-fonte modular sob responsabilidade de TI e mantido no diretório da automação dentro do monorepo.

### RNF-04 — Testabilidade
Regras de domínio e extratores devem possuir testes automatizados.

### RNF-05 — Performance
O projeto deve considerar lotes na ordem de aproximadamente 1.000 comprovantes/dia.

### RNF-06 — Segurança
A aplicação não deve depender de envio do conteúdo financeiro para serviços externos no MVP.

### RNF-07 — Observabilidade
Mensagens operacionais na GUI/CMD e log técnico opcional em modo debug.

### RNF-08 — Recuperação
Falhas individuais não devem invalidar comprovantes processados corretamente.

### RNF-09 — Isolamento no monorepo
A automação deve manter seu código, testes, documentação e configuração dentro de `separador_comprovantes_itau/`, evitando dependências com outras automações sem necessidade técnica explícita.

## 13. Fora do escopo inicial

- acesso automatizado ao Bankline;
- download automático dos comprovantes;
- OCR;
- uso obrigatório de IA;
- integração com Datasul/APB;
- anexação automática de comprovantes no ERP;
- auditoria formal de usuário;
- administração de permissões da pasta de rede.

Esses itens podem ser avaliados como evoluções.

## 14. Critérios de aceite do MVP

O MVP será considerado funcional quando:
1. receber um PDF Itaú válido;
2. identificar os comprovantes suportados;
3. separar comprovantes mesmo quando existirem múltiplos na página;
4. extrair data, favorecido e valor nos layouts homologados;
5. gerar PDFs individuais válidos;
6. aplicar a nomenclatura definida;
7. criar `AAAA/MM/DD` conforme data do pagamento;
8. preservar o original no histórico;
9. continuar o processamento diante de erro individual;
10. gerar lista de pendências;
11. apresentar erro de permissão quando não conseguir gravar;
12. executar por GUI e CMD;
13. passar pela suíte de testes definida para o MVP.
