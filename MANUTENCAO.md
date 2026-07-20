# Manutenção do site ahenriquecp.com

Guia de operação para o site pessoal acadêmico hospedado no GitHub Pages.

## Onde tudo mora

- **Domínio:** ahenriquecp.com (registrado na GoDaddy — só o registro; DNS também é gerenciado na GoDaddy).
- **Hospedagem:** GitHub Pages, gratuito.
- **Repositório:** https://github.com/andersonheri/andersonheri.github.io — repositório *público* de usuário. O nome `andersonheri.github.io` é obrigatório para o Pages funcionar como site pessoal na raiz.
- **Branch publicada:** `main`, pasta raiz (`/`). Cada push nessa branch dispara um novo deploy automático.
- **Arquivos principais:**
  - `index.html` — a página em si. HTML puro, sem dependências externas (sem CDN, sem fontes remotas, sem scripts).
  - `CNAME` — contém exatamente `ahenriquecp.com`. **Não apagar.** Se apagar, o custom domain some das configurações do Pages e o site volta a responder só em `andersonheri.github.io`.
  - `MANUTENCAO.md` — este arquivo.

## Editar o site

### Opção 1 — pelo terminal (recomendado)

```bash
cd ~/sites/ahenriquecp
# faça as alterações no index.html (qualquer editor)
git add index.html
git commit -m "Descrição curta da mudança"
git push
```

O deploy leva ~30–90 segundos. Confira em <https://ahenriquecp.com>.

### Opção 2 — direto no GitHub (para correções rápidas)

1. Abra <https://github.com/andersonheri/andersonheri.github.io/blob/main/index.html>.
2. Clique no ícone de lápis (Edit).
3. Faça a alteração, role até o fim, escreva a mensagem de commit e clique em **Commit changes**.

### Antes de fazer push, teste local

```bash
cd ~/sites/ahenriquecp
python3 -m http.server 8000
```

Abra <http://localhost:8000> no navegador. Ctrl+C encerra o servidor.

## Regras que evitam quebrar o site

1. **Nunca apagar o arquivo `CNAME`** nem alterar seu conteúdo. Se precisar mudar de domínio, altere no painel `Settings → Pages → Custom domain` e o GitHub reescreve o arquivo.
2. **Não incluir dependências externas** (fontes do Google, ícones de CDN, analytics de terceiros) sem necessidade. Elas quebram acessibilidade, criam vazamento de dados dos visitantes e podem ficar carregando lentamente.
3. **Nunca fazer `push --force` na `main`**. O GitHub Pages sempre publica o topo da main; um force push pode publicar uma versão errada.
4. **Não colocar nada sensível no repositório** — ele é público. Isso inclui rascunhos de artigos sob revisão, dados de pesquisa não publicados, e-mails privados etc.

## Verificar se o site está no ar

```bash
curl -I https://ahenriquecp.com
```

Deve retornar `HTTP/2 200`. Se retornar 404 ou 5xx logo depois de um deploy, aguarde 1–2 minutos e teste de novo.

Status detalhado do Pages:

```bash
gh api repos/andersonheri/andersonheri.github.io/pages
```

## DNS (raramente precisa mexer)

Painel: GoDaddy → Meus Produtos → Domínios → `ahenriquecp.com` → DNS.

Estes são os registros **que o site depende** — não apagar:

| Tipo  | Nome | Valor                       |
|-------|------|-----------------------------|
| A     | `@`  | `185.199.108.153`           |
| A     | `@`  | `185.199.109.153`           |
| A     | `@`  | `185.199.110.153`           |
| A     | `@`  | `185.199.111.153`           |
| CNAME | `www`| `andersonheri.github.io.`   |

Se o GitHub um dia atualizar essa lista (raro), a fonte oficial é: <https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site>.

Os demais registros (NS, SOA, TXT, DKIM da GoDaddy, `_acme-challenge`, `_domainconnect`) são independentes do site — não mexa neles sem saber o que fazem.

## HTTPS

O certificado é gerenciado automaticamente pelo GitHub via Let's Encrypt. Renova sozinho a cada ~60 dias. Só é preciso agir se:

- **Certificado não emite após 24h do DNS estar correto**: em `Settings → Pages`, clique em **Remove** no custom domain e depois recoloque `ahenriquecp.com`. Isso força uma nova requisição.
- **Enforce HTTPS aparece desligado**: em `Settings → Pages`, marque **Enforce HTTPS**. Só fica disponível depois que o certificado é emitido.

## Verificar propriedade do domínio (opcional, mas recomendado)

Evita que outra pessoa consiga apontar seu domínio para o GitHub Pages dela caso o repositório seja excluído. Fluxo:

1. Acesse <https://github.com/settings/pages> (nível do usuário, não do repositório).
2. Clique em **Add a domain** → digite `ahenriquecp.com` → **Add domain**.
3. O GitHub mostra um registro TXT do tipo `_github-pages-challenge-andersonheri` com um token. Copie o valor.
4. Na GoDaddy → DNS → **Adicionar novo registro**:
   - Tipo: `TXT`
   - Nome: `_github-pages-challenge-andersonheri`
   - Valor: cole o token
   - TTL: 1 hora
5. Volte no GitHub e clique em **Verify**. Depois de verificado, o registro TXT pode ser mantido; se remover, a verificação cai.

## Solução de problemas rápidos

| Sintoma | Provável causa | O que fazer |
|---------|---------------|-------------|
| Site volta ao "Coming Soon" da GoDaddy | Alguém adicionou um A `@` apontando para IP da GoDaddy | Apague esse A na GoDaddy. Só os 4 IPs do GitHub devem estar no apex. |
| `https://ahenriquecp.com` mostra erro de certificado | Cert ainda não emitido, ou expirou por bug | Aguarde 24h. Se persistir, remova e recoloque o custom domain em Settings → Pages. |
| Alterei o `index.html` e o site continua igual | Cache do navegador ou CDN do GitHub | Ctrl+Shift+R (recarga forçada). O cache do Pages é curto (~10 min). |
| `git push` reclama de autenticação | Token do gh CLI expirado | `gh auth login` |
| Site fora do ar em `andersonheri.github.io` mas ok no domínio próprio | Comportamento normal — com custom domain, o `.github.io` redireciona 301 para `ahenriquecp.com` | Nada a fazer. |

## Backup

O código-fonte é o próprio repositório do GitHub. Enquanto o repositório existir, você tem backup completo (incluindo histórico).

Para uma cópia local extra:

```bash
cd ~/sites/ahenriquecp
git pull
```
