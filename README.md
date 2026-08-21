# automatize_tarefas_macantes

Repositório de automações e soluções desenvolvidas para simplificar
tarefas repetitivas, reduzir atividades manuais e melhorar a
produtividade no dia a dia.

## Sobre o projeto

O `automatize_tarefas_macantes` nasce com uma ideia simples:

> Se uma tarefa é repetitiva, previsível e consome tempo sem agregar
> valor, vale avaliar se ela pode ser automatizada.

Este repositório reúne diferentes soluções de automação, principalmente
em Python, criadas para resolver problemas reais do cotidiano de
trabalho.

Cada solução deve buscar transformar processos manuais em fluxos mais
simples, padronizados, rastreáveis e fáceis de utilizar.

## Objetivos

-   Automatizar tarefas manuais e repetitivas.
-   Reduzir tempo gasto em atividades operacionais.
-   Diminuir riscos de erro humano.
-   Padronizar processos recorrentes.
-   Criar soluções reutilizáveis e de fácil manutenção.
-   Aplicar boas práticas de engenharia de software em automações.
-   Manter o conhecimento técnico e o código-fonte centralizados.
-   Permitir que soluções técnicas sejam disponibilizadas aos usuários
    de forma simples.

## Princípios

As soluções deste repositório devem, sempre que fizer sentido:

-   resolver um problema real;
-   possuir escopo e regras de negócio documentados;
-   separar regras de negócio de detalhes técnicos;
-   ser modulares e testáveis;
-   possuir tratamento adequado de erros;
-   gerar mensagens claras para o usuário;
-   evitar dependências desnecessárias;
-   considerar segurança e privacidade dos dados;
-   permitir evolução e manutenção pelo time técnico.

A complexidade da arquitetura deve ser proporcional ao problema. Nem
toda automação precisa de DDD ou Clean Architecture, mas soluções com
regras de negócio relevantes devem ser estruturadas para permanecer
compreensíveis e sustentáveis ao longo do tempo.

## Estrutura do repositório

Cada automação deve ser tratada como uma solução independente dentro do
repositório.

Estrutura inicial sugerida:

``` text
automatize_tarefas_macantes/
│
├── README.md
│
├── separador_comprovantes_itau/
│   ├── README.md
│   ├── docs/
│   ├── src/
│   └── tests/
│
└── futuras_automacoes/
```

A estrutura interna poderá variar conforme a necessidade de cada
solução.

## Soluções

### Separador de Comprovantes Itaú

Automação para processamento de PDFs consolidados de comprovantes
bancários.

A solução permite selecionar um PDF extraído do Itaú, identificar os
comprovantes existentes, separar os documentos, extrair informações
relevantes e organizar automaticamente os arquivos.

Principais funcionalidades previstas:

-   leitura de PDF consolidado;
-   identificação de diferentes layouts de comprovantes;
-   separação de múltiplos comprovantes;
-   extração de data, favorecido e valor;
-   geração de PDFs individuais;
-   nomenclatura padronizada;
-   organização automática por ano, mês e dia;
-   tratamento de pendências;
-   histórico de processamento;
-   modo debug;
-   interface gráfica para Windows;
-   execução via linha de comando.

Documentação específica da solução deve permanecer dentro do próprio
diretório da feature.

## Tecnologias

O repositório poderá utilizar diferentes tecnologias conforme cada
problema.

Python será uma das principais linguagens utilizadas devido à sua
aderência a:

-   automação;
-   manipulação de arquivos;
-   processamento de dados;
-   integração com APIs;
-   documentos e PDFs;
-   sistemas corporativos;
-   interfaces simples;
-   inteligência artificial.

A escolha tecnológica deve sempre considerar o problema antes da
ferramenta.

## Qualidade

Sempre que aplicável, as soluções devem possuir:

``` text
src/        Código-fonte
tests/      Testes automatizados
docs/       Documentação técnica e funcional
README.md   Visão geral da solução
```

Correções de bugs relevantes devem, preferencialmente, resultar em
testes de regressão.

## Segurança

Dados corporativos, financeiros ou pessoais não devem ser versionados no
repositório.

Isso inclui:

-   arquivos reais de produção;
-   comprovantes bancários;
-   credenciais;
-   tokens;
-   senhas;
-   chaves de API;
-   arquivos de configuração contendo segredos.

Arquivos utilizados em testes devem utilizar dados fictícios ou
anonimizados.

## Evolução

Este repositório não representa uma única aplicação.

A ideia é formar progressivamente um catálogo de pequenas soluções
capazes de eliminar tarefas maçantes do cotidiano.

Uma nova automação pode começar pequena:

``` text
Problema manual
      ↓
Automação
      ↓
Padronização
      ↓
Testes
      ↓
Uso no dia a dia
      ↓
Evolução
```

O objetivo não é automatizar por automatizar.

O objetivo é usar tecnologia para retirar esforço de atividades
repetitivas e liberar tempo para trabalhos que realmente exigem análise,
criatividade e tomada de decisão.
