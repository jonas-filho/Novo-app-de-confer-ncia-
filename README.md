# Conferência de pedidos (PWA)

App para conferir mercadoria bipando o código de barras pela câmera, a partir do DAV em PDF.
Sem build e sem dependências externas: o pdf.js e o ZXing já vão na pasta `vendor/`, então funciona offline.

## Como rodar
- Local: `npm start` (ou `python3 -m http.server 8080`) e abra `http://localhost:8080`.
- Celular: publique a pasta em qualquer hospedagem **HTTPS** (GitHub Pages, Netlify, Vercel). A câmera só funciona em HTTPS ou localhost.
- Testes: `npm test`.

## Fluxo
1. **Importar pedido (PDF)** → o app lê o DAV e monta a lista de itens e quantidades.
2. **Abrir câmera** e bipar (ou digitar / usar leitor USB-Bluetooth). Cada leitura soma 1 no item, com bip e vibração.
3. Código que não está no pedido: o app pergunta a qual item corresponde e **lembra** para os próximos pedidos.
4. **Fechar volume** → informa o peso (kg) → imprime ou envia a relação dos itens daquele volume. Segue conferindo o resto.
5. Em **Volumes**: reimprimir, enviar, reabrir o último volume, ver pendências e **Finalizar pedido**.

## Estrutura
```
index.html, manifest.webmanifest, sw.js
css/app.css
js/
  main.js                 roteador (#/, #/check/ID, #/volumes/ID)
  config.js               padrões do DAV (regex) e opções do leitor  ← ajuste o PDF aqui
  domain/order.js         regras: bipar, excesso, volumes, progresso (puro, testado)
  services/
    pdfReader.js          PDF → linhas de texto
    davParser.js          linhas → cabeçalho + itens
    scanner.js            câmera (BarcodeDetector nativo, ZXing como reserva)
    repository.js         persistência (localStorage)  ← troque por API quando quiser
    report.js             textos, WhatsApp/compartilhar, impressão
  ui/ dom.js, dialog.js, toast.js, feedback.js, components/dialogs.js, views/{home,check,volumes}.js
vendor/                   pdf.js e ZXing (locais)
tests/order.test.js
```

## Manutenção
- **PDF não reconhecido?** O app mostra o texto extraído. Ajuste `DAV_PATTERNS` em `js/config.js` (grupos `code`, `desc`, `qty`, `unit`) e rode `npm test`.
- **Atualizou algum arquivo?** Aumente `CACHE_VERSION` em `sw.js`, senão o celular continua com a versão antiga em cache.
- **Vários aparelhos, painel do administrador, acesso do financeiro:** exige um servidor. Só `js/services/repository.js` precisa mudar; telas e regras continuam iguais.
