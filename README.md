# Modelagem — Visualização de Treinamento / Trilha

Protótipos estáticos (HTML/CSS/JS puro, sem build) das telas de trilha de
treinamento do Suporte Lector.

## Telas

| Arquivo | Tela |
| --- | --- |
| `trilha.html` | Trilha Completa Automação — visão da trilha, com etapas, conteúdos e abas Descrição / Desempenho / Relatórios. É a tela de entrada do fluxo. |
| `index.html` | Treinamento Skill Suporte Lector — tela de um conteúdo, com sidebar, leitor de PDF/vídeo inline, fórum, chat com tutor e gráfico de desempenho. Abre em overlay a partir da trilha. |
| `avaliacao.html` | Avaliação com 5 questões (única, múltipla, V/F, associativa e lacuna), cronômetro e tela de resultado. Abre a partir da trilha ou do treinamento. |
| `certificado.html` | Certificado de conclusão. |
| `Trilha Desempenho.html` | Aba de desempenho isolada (variante de design). |
| `Trilha Wireframes.html` | Wireframes de baixa fidelidade da trilha. |

## Rodando localmente

Não há `package.json`, lint, build ou testes — é um site estático:

```bash
npx serve -p 3131 .
```

Depois abra <http://localhost:3131/trilha.html>. Para validar mudanças, abra a
tela no navegador — não existe suíte automatizada.

## Deploy

Deploy contínuo na Vercel a partir da branch `main` deste repositório
(projeto `modelagem-visualizacao-treinamento-trilha`). `vercel.json` liga
`cleanUrls`, então as URLs em produção não têm `.html`
(ex.: `/trilha`, `/avaliacao`).
