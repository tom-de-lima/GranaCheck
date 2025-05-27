**Documentação do App “Calculadora de Remuneração”**

---

## 1. Visão Geral

Este aplicativo web calcula a remuneração de servidores públicos, reunindo:

* **Gratificações** (risco de vida, adicional noturno, horas excedentes, escolaridade, quinquênio, especialização e extras);
* **Auxílio Alimentação** (base e extras);
* **Descontos** (previdenciário, sindicato, imposto de renda, dependentes e demais descontos individuais);
* **Temas** (claro e escuro) com troca dinâmica de paleta.

Ele foi construído com **HTML**, **CSS** e **JavaScript** puro, seguindo práticas de modularização e design responsivo.

---

## 2. Estrutura de Arquivos

```
remuneracao-app/
├── index.html       ← marca o layout e captura das entradas
├── css/
│   └── styles.css   ← estilos, variáveis de tema e dark mode
└── js/
    └── app.js       ← lógica de máscara, cálculos, renderização e troca de tema
```

---

## 3. Descrição dos Arquivos

### 3.1 index.html

* **Cabeçalho (`<header>`)**

  * Título do app e botão **Modo Noturno/Claro** (`#toggle-theme`).
* **Formulário (`<form id="remuneracao-form">`)**

  * **Base Atual**: `<select id="baseAtual">` com apenas duas opções (2.083,54 ou 2.003,40).
  * **Demais campos**: plantões, escolaridade, quinquênio, especialização, extras (24h, 10h diurno, 10h noturno, festivos), valor extra festivo, dependentes, sindicalizado, descontos individuais.
  * **Botão** `<button type="submit">Calcular</button>`.
* **Seção de Resultados** (`<section id="resultados">`)

  * Recebe, via JavaScript, `<div class="item">Label – Valor</div>`.
* **Rodapé (`<footer>`)**

  * Créditos e contatos.

### 3.2 css/styles.css

* **Variáveis CSS** (`:root`) para cores e espaçamentos (modo claro).
* **`.dark-theme`** redefine todas as variáveis para paleta noturna.
* **Reset básico** (`* { box-sizing; margin; padding }`).
* **Estilos de layout**

  * `.container`, `.header`, `form`, `.field`, `button`, `#resultados .item`, `.footer`.
  * Utilização de `var(--nome-da-variavel)` para cores, bordas, fundos e transições.
* **Media query** para telas < 360px.

### 3.3 js/app.js

1. **Máscaras de moeda**

   * `aplicarMascaraMoeda(e)`: formata qualquer `<input>` registrado (base, extra festivo e outros descontos) via `Intl.NumberFormat('pt-BR', ...)`.
2. **Parser BRL → Number**

   * `parseBRL("2.083,54") //→ 2083.54`
3. **Evento de submit**

   * Lê valores de todos os campos do formulário;
   * Executa **todos** os cálculos em sequência, conforme especificações;
   * Monta a lista `itens = [ { label, value }, … ]`;
   * Soma `totalBruto`, calcula `previdencia`, `sindicato`, `valorDependentes`, `descontoIR` e `totalLiquido`;
   * Renderiza dinamicamente cada `<div class="item">` em `#resultados`, incluindo destaques:

     * `.highlight-base` para “Base Atual”,
     * `.total-liquido` para o valor líquido,
     * `.total-descontos` para o somatório de descontos.
4. **Modo Noturno**

   * Botão `#toggle-theme` adiciona/ remove a classe `dark-theme` ao `<html>`;
   * O texto do botão alterna entre “Modo Noturno” e “Modo Claro”.

---

## 4. Fluxo de Funcionamento

1. **Carregamento da página**

   * `DOMContentLoaded` registra máscaras e eventos.
2. **Entrada de dados**

   * Usuário seleciona e digita valores no formulário;
   * Máscara converte em tempo real qualquer valor monetário para “R\$ x.xxx,yy”.
3. **Envio do formulário**

   * JS coleta cada campo, usa `parseBRL` onde necessário;
   * Executa cálculos definidos;
   * Atualiza o DOM em `#resultados`.
4. **Troca de tema**

   * Clique no botão altera apenas variáveis CSS, sem recarregar a página.

---

## 5. Exemplos de Interação via Console

No **Console do navegador**, você pode testar partes do script sem submeter o formulário:

```js
// 1. Converter string BRL para número:
parseBRL("2.083,54");         // → 2083.54
parseBRL("1.234,56");         // → 1234.56

// 2. Testar máscara de moeda (simulando evento):
const fakeEvent = { target: { value: "123456", /* sem máscara*/ } };
aplicarMascaraMoeda(fakeEvent);
console.log(fakeEvent.target.value); // → "1.234,56"

// 3. Obter valores já inseridos no form:
document.getElementById('baseAtual').value;        // e.g. "2.083,54"
document.getElementById('dependentes').value;     // e.g. "2"

// 4. Após o usuário clicar em “Calcular”, inspecione variáveis internas:
form.addEventListener('submit', e => {
  e.preventDefault();
  console.log({ baseAtual, totalBruto, totalLiquido, descontoIR });
});
```

---

## 6. Exemplo de Caso de Uso Completo

1. Usuario seleciona:

   * **Base Atual**: 2.083,54
   * **Plantões**: 7
   * **Escolaridade**: Graduação
   * **Quinquênio**: 1
   * **Especialização**: 15%
   * **Extras**: 1 x 24h, 0 x 10h diurno, 0 x 10h noturno, 0 festivos
   * **Valor Extra Festivo**: 0
   * **Dependentes**: 2
   * **Sindicalizado**: Sim
   * **Descontos Individuais**: R\$ 100,00

2. Clique em **Calcular**.

3. **Saída esperada** (`#resultados`):

| Item                          | Valor             |
| ----------------------------- | ----------------- |
| Base Inicial                  | R\$ 1.890,00      |
| **Base Atual** (highlight)    | R\$ 2.083,54      |
| Risco de Vida                 | R\$ 3.125,31      |
| Adicional Noturno             | R\$ 321,21        |
| Horas Excedentes 50%          | R\$ 332,06        |
| Horas Excedentes 70%          | R\$ 332,07        |
| Auxílio Alimentação           | R\$ 793,80        |
| Serviço Extra 24h             | R\$ 370,00        |
| Serviço Extra 10h Diurno      | R\$ 0,00          |
| Serviço Extra 10h Noturno     | R\$ 0,00          |
| Extras Festivos (total)       | R\$ 0,00          |
| Escolaridade                  | R\$ 208,35        |
| Quinquênio                    | R\$ 104,18        |
| Especialização                | R\$ 1.434,08      |
| **Total Bruto**               | **R\$ 10.994,60** |
| **Total Líquido** (green bg)  | **R\$ 9.218,80**  |
| Sindicato                     | R\$ 41,67         |
| Santa Cruz Prev (previdência) | R\$ 306,28        |
| Imposto de Renda              | R\$ 1.327,85      |
| Descontos Individuais         | R\$ 100,00        |
| **Total Descontos** (red bg)  | **R\$ 1.775,80**  |

*Todos os valores são formatados pelo `Intl.NumberFormat('pt-BR', ...)`.*

---

## 7. Como Funciona a Integração

* **HTML** define IDs e estruturas para o JS manipular (captura de entradas e ponto de inserção dos resultados).
* **CSS** utiliza variáveis de tema que podem ser trocadas em runtime via a classe `.dark-theme` no elemento raiz.
* **JS** amarra tudo:

  1. Registra máscaras em inputs,
  2. Converte strings para números,
  3. Executa cálculos exatos em JavaScript,
  4. Monta a exibição dinâmica de resultados,
  5. Controla o toggle de tema sem recarregar a página.

---
