# desafio-DIO
# Simulador de Fundos Imobiliários

Ferramenta em Excel que simula investimentos em fundos imobiliários (FIIs): do aporte de cada mês até os dividendos que ele rende. A planilha foi pensada para ser usada por qualquer pessoa, sem precisar saber fórmula: basta preencher os campos amarelos e o restante é calculado.

Arquivo: `simulador_fundos_imobiliarios.xlsx` (abas **Simulador** e **Apoio**).

> Todos os valores do arquivo (salário, taxa e percentuais) são **exemplos ilustrativos**. A ferramenta é uma simulação e não é promessa de retorno nem recomendação de investimento.

## Como usar

1. Na aba **Simulador**, preencha os campos amarelos (texto azul): salário, rendimento mensal da carteira, percentual do salário a investir e prazo em anos.
2. Escolha o perfil de investidor na lista suspensa.
3. Leia os resultados nos campos cinza, que são calculados pela planilha e não devem ser editados.

## Perguntas que a ferramenta responde

Valores obtidos com os dados de exemplo (salário de R$ 5.000, rendimento de 0,8% ao mês, 30% do salário, 10 anos):

| Pergunta de negócio | Onde aparece | Exemplo |
|---|---|---|
| Quanto investir por mês | `Simulador!C13` | R$ 1.500,00 |
| Por quantos anos | `Simulador!C14` | 10 anos |
| Qual a taxa de rendimento mensal | `Simulador!C9` | 0,80% |
| Quanto de patrimônio vai acumular | `Simulador!C15` | R$ 300.326,20 |
| Quanto vai receber de dividendos por mês | `Simulador!C18` | R$ 2.402,61 |

Além das cinco respostas, a planilha mostra:

- **Quanto saiu do seu bolso** (`C16`) e **quanto veio dos rendimentos** (`C17`), para separar o que foi aportado do que foi gerado pelos retornos.
- Uma **frase-resumo** (`B19`) que monta o resultado em texto e muda junto com os números.
- Uma **projeção de cenários** (`B22:E27`) com 2, 5, 10, 20 e 30 anos extras.
- A **divisão do aporte** (`B31:D38`) entre seis tipos de fundo, conforme o perfil escolhido.

## Como o VF e o PROCV entram nos cálculos

### VF (valor futuro)

O patrimônio acumulado vem da função VF:

```
=VF(taxa_mensal; anos*12; -aporte)
```

- `taxa_mensal`: rendimento por mês, na mesma unidade dos aportes.
- `anos*12`: número de períodos. Como os aportes são mensais, o prazo em anos é convertido em meses.
- `-aporte`: valor aportado a cada mês, com sinal negativo porque o Excel trata o dinheiro que sai do bolso como saída. Sem o sinal, o patrimônio apareceria negativo.
- Os argumentos opcionais (valor presente e tipo) ficam em branco: a simulação parte do zero e o aporte é feito no fim de cada mês.

A partir do patrimônio, os dividendos mensais são calculados como `=patrimonio*taxa_mensal`, ou seja, o rendimento de um mês sobre tudo o que foi acumulado.

Nos cenários, a mesma lógica é repetida para cada prazo extra da coluna B:

- Total investido: `=aporte*B23*12+C16`
- Patrimônio: `=VF(taxa_mensal; B23*12; -aporte)+patrimonio`
- Dividendos mensais: `=D23*taxa_mensal`

Cada cenário soma ao patrimônio da simulação principal o valor acumulado pelos aportes dos anos extras. O patrimônio já acumulado entra somado, sem reaplicação adicional ao longo desses anos.

### PROCV com chave composta

O percentual de cada tipo de fundo depende de **dois** critérios: o perfil e o tipo de fundo. Como o PROCV procura um único valor na primeira coluna da tabela, a aba **Apoio** tem uma coluna de chave que junta os dois:

```
=B5&"|"&C5        →   Moderado|Logística
```

Na aba Simulador, a mesma chave é montada com o perfil escolhido e o tipo da linha:

```
=PROCV(perfil&"|"&B32; tabela_perfis; 4; FALSO)
```

- `perfil&"|"&B32`: valor procurado (por exemplo, `Moderado|Logística`).
- `tabela_perfis`: intervalo da aba Apoio, em que a chave é a primeira coluna.
- `4`: coluna do intervalo que devolve o percentual.
- `FALSO`: exige correspondência exata.

O valor destinado a cada tipo é `=aporte*percentual`. Ao trocar o perfil na lista, a divisão do aporte muda sozinha.

## Intervalos nomeados

As fórmulas usam nomes em vez de endereços de célula, o que as deixa mais legíveis e evita erros ao copiar.

| Nome | Referência | O que guarda |
|---|---|---|
| `salario` | `Simulador!C8` | Salário mensal |
| `taxa_mensal` | `Simulador!C9` | Rendimento mensal da carteira |
| `perc_sugestao` | `Simulador!C10` | Percentual do salário a investir |
| `aporte` | `Simulador!C13` | Aporte mensal (`salario*perc_sugestao`) |
| `anos` | `Simulador!C14` | Prazo da simulação, em anos |
| `patrimonio` | `Simulador!C15` | Patrimônio acumulado |
| `dividendos_mensais` | `Simulador!C18` | Dividendos mensais |
| `perfil` | `Simulador!C30` | Perfil de investidor escolhido |
| `tabela_perfis` | `Apoio!A5:D22` | Tabela de chave, perfil, tipo e percentual |
| `lista_perfis` | `Apoio!F5:F7` | Perfis disponíveis na lista suspensa |

## Percentuais de cada perfil

A divisão do aporte entre os seis tipos de fundo, por perfil. Cada perfil soma 100%.

| Tipo de fundo | Conservador | Moderado | Arrojado |
|---|---|---|---|
| Lajes Corporativas | 10% | 15% | 20% |
| Logística | 20% | 20% | 25% |
| Shoppings | 10% | 20% | 25% |
| Papéis (CRI) | 40% | 25% | 5% |
| Fundos de Fundos (FoF) | 15% | 10% | 10% |
| Híbridos | 5% | 10% | 15% |
| **Total** | **100%** | **100%** | **100%** |

**Origem:** são percentuais **ilustrativos**, definidos para este exemplo. Não vêm de uma carteira real, de um estudo ou de uma recomendação de investimento. A lógica segue a ideia de que perfis mais conservadores concentram mais em papéis (CRI) e perfis mais arrojados em fundos de tijolo, como lajes, logística e shoppings. Quem for usar a ferramenta pode trocar os percentuais na aba Apoio.

A aba Apoio tem um quadro de conferência (`G5:H7`) que soma cada perfil com `SOMASE` e mostra "Tudo certo" quando o total é 100%. Na aba Simulador, a linha de total faz a mesma checagem.

## O que mudei em relação à ferramenta do Expert

Além do que o desafio pede, acrescentei:

- **Separação entre o que saiu do bolso e o que veio dos rendimentos**, para mostrar quanto do patrimônio é fruto dos retornos.
- **Frase-resumo dinâmica**, que traduz o resultado em uma frase simples.
- **Cenários de "tempo extra"**, que somam anos de aportes ao patrimônio da simulação principal.
- **Aporte calculado a partir do salário**, usando o percentual escolhido (`salario*perc_sugestao`), em vez de um valor digitado.
- **Validação de dados nas entradas**: salário maior ou igual a zero, taxa e percentual entre 0% e 100%, prazo entre 1 e 60 anos e perfil escolhido apenas pela lista.
- **Quadro de conferência de 100%** na aba Apoio.
- **Linguagem mais próxima e direta**, com textos de apoio ao lado de cada campo e uma legenda de cores (amarelo e texto azul para editar, cinza para calculado).



### Perfil Conservador

<img width="856" height="214" alt="image" src="https://github.com/user-attachments/assets/0143dde2-61a5-4e9c-a3c8-bd998beeff70" />


### Perfil Arrojado

<img width="856" height="214" alt="image" src="https://github.com/user-attachments/assets/646765a4-0e50-4acc-bd83-2ce061b43327" />


## Limitações

- A simulação usa uma taxa de rendimento mensal **constante**, sem considerar impostos, taxas de administração, variação de cotas nem inflação.
- Os dividendos mensais são uma estimativa simples (`patrimônio × taxa mensal`).
- Rentabilidade passada não garante rentabilidade futura, e os resultados são projeções.
