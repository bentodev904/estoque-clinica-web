# 🧩 Estoque Clínica

Projeto de sistema de controle de estoque com QR Code para a clínicas de desenvolvimento infantil.

## Deploy em 5 minutos

## Pré-requisitos
- VM ARM já rodando Ubuntu
- SSH configurado
- IP público da VM em mãos

---

## 1. Abrir porta no Oracle Cloud (painel web)

Oracle bloqueia tudo por padrão no Security List.

1. Acessa **cloud.oracle.com → Networking → Virtual Cloud Networks**
2. Clica na sua VCN → **Security Lists → Default Security List**
3. **Add Ingress Rule:**
   - Source CIDR: `0.0.0.0/0`
   - IP Protocol: `TCP`
   - Destination Port: `8000`
4. Salva

---

## 2. Enviar arquivos pro servidor

No seu PC (ou Termux), dentro da pasta `backend/`:

```bash
scp main.py requirements.txt deploy.sh ubuntu@<SEU_IP_PUBLICO>:~
```

> Substitui `ubuntu` pelo seu usuário se for diferente.

---

## 3. Rodar o deploy

```bash
ssh ubuntu@<SEU_IP_PUBLICO>
bash deploy.sh
```

O script faz automaticamente:
- Instala Python 3 + venv
- Instala FastAPI e uvicorn
- Cria serviço systemd (inicia no boot)
- Libera porta 8000 no iptables interno

---

## 4. Verificar se subiu

```bash
# Status do serviço
sudo systemctl status clinica

# Testar localmente na VM
curl http://localhost:8000/api/items

# Testar de fora
curl http://<SEU_IP_PUBLICO>:8000/api/items
```

Deve retornar `[]`.

---

## 5. Configurar o frontend

Abre o `frontend/index.html` no navegador.  
No banner amarelo, cola:

```
http://<SEU_IP_PUBLICO>:8000
```

Clica **Salvar** — pronto, todos os dispositivos que usarem o mesmo HTML e a mesma URL verão os dados em tempo real.

---

## Comandos úteis no servidor

```bash
# Ver logs em tempo real
sudo journalctl -u clinica -f

# Reiniciar após atualizar main.py
sudo systemctl restart clinica

# Atualizar o código
scp main.py ubuntu@<IP>:/opt/clinica/main.py
ssh ubuntu@<IP> "sudo systemctl restart clinica"
```

## Estrutura

```
backend/
  main.py          # API FastAPI (SQLite)
  requirements.txt
  Procfile
  railway.toml

frontend/
  index.html       # App completo (abre em qualquer navegador)
```

## Endpoints da API

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | /api/items | Listar itens |
| POST | /api/items | Criar item |
| PUT | /api/items/{id} | Editar item |
| DELETE | /api/items/{id} | Remover item |
| GET | /api/loans | Listar empréstimos |
| POST | /api/loans | Registrar empréstimo |
| POST | /api/loans/{id}/return | Registrar devolução |
| GET | /docs | Swagger UI automático |
