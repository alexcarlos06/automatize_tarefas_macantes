# Arquitetura — Separador de Comprovantes Itaú

## 1. Diretriz

O projeto utilizará princípios de:
- Domain-Driven Design (DDD) de forma pragmática;
- Clean Architecture;
- SOLID;
- separação de responsabilidades;
- Strategy Pattern para layouts;
- Dependency Inversion para infraestrutura.

DDD será usado para tornar explícitas as regras do processamento, não para adicionar complexidade desnecessária.

## 2. Contexto delimitado

O MVP possui um bounded context principal:

**Processamento de Comprovantes Bancários**

Responsabilidades:
- identificar comprovantes;
- interpretar seus dados;
- validar regras;
- gerar nomenclatura;
- definir destino;
- registrar resultado.

Bankline e sistema de arquivos são detalhes externos ao domínio.

## 3. Camadas

```text
Interface (GUI / CLI)
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

Não deve depender de:
- Tkinter/CustomTkinter;
- PyMuPDF;
- sistema operacional;
- arquivos Excel;
- logging concreto.

Possíveis objetos:
- `Comprovante`;
- `DadosComprovante`;
- `DataPagamento`;
- `Valor`;
- `Favorecido`;
- `IdentificadorDiario`;
- `StatusProcessamento`;
- `Pendencia`.

### Application

Casos de uso sugeridos:
- `ProcessarArquivo`;
- `IdentificarComprovantes`;
- `GerarComprovantes`;
- `RegistrarHistorico`.

Orquestra domínio e portas de infraestrutura.

### Infrastructure

Implementações:
- leitor/manipulador PDF;
- extratores Itaú;
- sistema de arquivos;
- histórico;
- logging;
- empacotamento/integrações de SO.

### Interface

Adaptadores:
- GUI Windows;
- CLI/CMD.

GUI e CLI devem chamar o mesmo caso de uso.

## 4. Estratégia para layouts

Contrato conceitual:

```python
class ExtratorComprovante:
    def suporta(self, texto: str) -> bool:
        ...

    def extrair(self, comprovante_bruto):
        ...
```

Implementações:
- `ExtratorTransferenciaPix`;
- `ExtratorTed`;
- `ExtratorTransferenciaContas`;
- `ExtratorTituloOutrosBancos`;
- `ExtratorTituloItau`.

Um registry/factory selecionará a estratégia adequada.

## 5. Separação visual do PDF

Separar texto e separar visualmente o PDF são problemas relacionados, mas distintos.

O componente de PDF deve ser capaz de:
1. extrair texto e suas coordenadas;
2. localizar os limites lógicos do comprovante;
3. mapear os limites para regiões físicas;
4. gerar um PDF individual preservando a aparência do comprovante.

Isso é especialmente importante quando existem dois ou mais comprovantes na mesma página.

## 6. Portas sugeridas

```text
PdfReader
PdfSplitter
ComprovanteExtractor
FileRepository
HistoryRepository
ExecutionLogger
ProgressReporter
```

Os casos de uso dependem das abstrações; infraestrutura fornece implementações.

## 7. Estrutura sugerida

```text
src/
└── separador_comprovantes/
    ├── domain/
    │   ├── entities/
    │   ├── value_objects/
    │   ├── services/
    │   └── exceptions/
    ├── application/
    │   ├── use_cases/
    │   ├── ports/
    │   └── dto/
    ├── infrastructure/
    │   ├── pdf/
    │   ├── extractors/
    │   ├── filesystem/
    │   ├── history/
    │   └── logging/
    ├── presentation/
    │   ├── gui/
    │   └── cli/
    └── bootstrap.py
tests/
├── unit/
├── integration/
├── regression/
└── fixtures/
```

## 8. Dependências

Escolhas iniciais recomendadas:
- Python 3.12 ou versão corporativamente homologada;
- PyMuPDF para leitura, coordenadas, recorte e geração;
- `pytest` para testes;
- biblioteca padrão `logging`;
- `argparse` ou Typer para CLI;
- Tkinter ou CustomTkinter para GUI;
- PyInstaller para distribuição Windows.

A seleção definitiva deve ocorrer no início da implementação e ser registrada como ADR quando relevante.

## 9. Configuração

Configurações técnicas não sensíveis podem ficar externas ao executável quando isso facilitar suporte.

Não colocar regras essenciais de negócio em configurações arbitrariamente editáveis pelo usuário.

## 10. Exceções

Separar:
- exceções de domínio;
- exceções de aplicação;
- exceções de infraestrutura.

Exemplos:
- `ComprovanteInvalido`;
- `LayoutNaoSuportado`;
- `CampoObrigatorioAusente`;
- `FalhaLeituraPdf`;
- `SemPermissaoGravacao`.

A camada de apresentação traduz exceções para mensagens apropriadas.

## 11. Concorrência

O MVP pode iniciar com processamento sequencial.

Antes de introduzir paralelismo, medir:
- tempo de leitura;
- tempo de recorte;
- I/O na rede;
- consumo de memória.

A integridade e previsibilidade são prioritárias em relação à otimização prematura.

## 12. Evoluções previstas

A arquitetura deve permitir futuramente:
- novos bancos;
- novos layouts Itaú;
- integração Datasul/APB;
- anexação automática;
- processamento em lote;
- automação de entrada;
- serviço/API;
- interface web.

Essas possibilidades não devem ampliar o escopo do MVP.
