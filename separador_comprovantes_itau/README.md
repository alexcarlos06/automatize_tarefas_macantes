# Separador de Comprovantes Itaú

Aplicação interna para automatizar a separação, identificação, renomeação e organização de comprovantes extraídos em PDF do Bankline Itaú.

## Objetivo

Substituir o processo manual em que o usuário:
1. baixa um PDF consolidado do Itaú;
2. identifica cada comprovante;
3. separa os comprovantes em PDFs individuais;
4. renomeia cada arquivo;
5. organiza os arquivos por data de pagamento.

A aplicação será distribuída ao usuário final como executável Windows, mantendo o código-fonte sob responsabilidade do time de TI.

## Documentação

- `docs/ESPECIFICACAO.md` — escopo funcional e requisitos do produto.
- `docs/REGRAS_DE_NEGOCIO.md` — regras de identificação, nomenclatura, diretórios, pendências e histórico.
- `docs/ARQUITETURA.md` — decisões de arquitetura, DDD, Clean Architecture e componentes.
- `docs/ENGENHARIA_DE_SOFTWARE.md` — padrões de desenvolvimento, testes, build, versionamento e qualidade.

## Interfaces previstas

- Interface gráfica Windows.
- Linha de comando (CMD), reutilizando o mesmo núcleo da aplicação.

## Status

Especificação inicial para desenvolvimento do MVP.
