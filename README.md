# IPS Patcher

Ferramenta web para aplicar patches de tradução em ROMs de jogos retrô. Roda 100% no navegador — nenhum arquivo é enviado a servidor.

## Como usar

1. Abra o `ips-patcher.html` no navegador (ou acesse via GitHub Pages)
2. Selecione a ROM do jogo
3. Selecione o patch `.ips`
4. Clique em **Aplicar patch**
5. O download do `.zip` com a ROM traduzida inicia automaticamente

---

## ROMs suportadas

| Formato | Console |
|--------|---------|
| `.gba` | Game Boy Advance |
| `.nes` | Nintendo Entertainment System |
| `.sfc` / `.smc` | Super Nintendo |
| `.gb` / `.gbc` | Game Boy / Game Boy Color |
| `.nds` | Nintendo DS |
| `.zip` | Qualquer um dos acima compactado |

> A ROM pode ser entregue compactada em `.zip`. O patcher extrai automaticamente o primeiro arquivo encontrado dentro do zip.

---

## Patches suportados

| Formato | Suporte |
|--------|---------|
| `.ips` | ✅ Suportado (incluindo registros RLE) |
| `.ups` | ❌ Não suportado |
| `.bps` | ❌ Não suportado |
| `.xdelta` | ❌ Não suportado |

---

## Limitações

- **Apenas IPS:** UPS, BPS e xdelta não são suportados. Esses formatos possuem verificação de checksum e algoritmos mais complexos.
- **Um arquivo por zip:** Quando a ROM vem em `.zip`, apenas o primeiro arquivo é extraído. Zips com múltiplos arquivos podem gerar resultado inesperado.
- **Sem validação de compatibilidade:** O patcher não verifica se o patch é compatível com a ROM fornecida. Patch errado = ROM corrompida.
- **Sem suporte a `.7z` ou `.rar`:** Apenas `.zip` com compressão `Stored` ou `Deflate` (padrão).
- **Memória do navegador:** ROMs muito grandes (acima de ~512 MB) podem causar travamento dependendo do dispositivo.
- **Sem verificação de CRC:** Não há checagem de integridade da ROM antes ou depois do patch.

---

## Para programadores

O projeto é um único arquivo HTML sem dependências externas.

**Algoritmo IPS** (`applyIPS`): lê registros sequencialmente a partir do offset 5 (após o header `PATCH`) até encontrar o marcador `EOF`. Cada registro tem:
- 3 bytes de offset (big-endian)
- 2 bytes de tamanho
- Se tamanho = 0 → registro RLE: 2 bytes de tamanho + 1 byte a repetir
- Caso contrário → copia N bytes no offset indicado

**Extração de ZIP** (`extractRomFromZip`): parser minimal que localiza a assinatura `PK\x03\x04`, lê o header local e descomprime via `DecompressionStream('deflate-raw')` (API nativa dos browsers modernos).

**Geração de ZIP** (`buildZip`): monta um ZIP válido com método `Stored` (sem compressão), calcula CRC-32 manualmente e escreve local header + central directory + EOCD.

### Adicionar suporte a UPS/BPS

Para suportar UPS ou BPS seria necessário:
- Ler o CRC32 da ROM original embutido no patch e validar antes de aplicar
- Implementar o algoritmo de diff reverso (BPS usa codificação mais sofisticada)
- UPS é o mais simples dos dois para implementar como próximo passo

---

## Compatibilidade

Funciona em qualquer browser moderno com suporte a `DecompressionStream` (Chrome 80+, Firefox 113+, Safari 16.4+). Não funciona no Internet Explorer.