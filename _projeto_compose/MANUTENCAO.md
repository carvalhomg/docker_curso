### ✅ MANUAL DE MANUTENÇÃO — Ambiente Tomcat Docker
##  📁 Estrutura Atual
tomcat-a → porta 8081\
tomcat-b → porta 8082\
Pastas separadas: apps-a/, apps-b/, logs-a/, logs-b/ context-config/ apache-tomcat-7.0.47/ \
Arquivos: Dockerfile e dockercompose.yaml \

|Objetivo do Projeto |
|---------------------|
>** Construímos um ambiente com Docker e Docker Compose, no qual criamos uma imagem do Tomcat 7. A partir dessa mesma imagem, subimos dois serviços em portas diferentes, executando as mesmas aplicações de forma redundante. Esse ambiente foi projetado para permitir atualizações ou a implantação de novas aplicações sem a necessidade de parar os serviços ativos. Mesmo que seja necessário interromper um dos serviços, a redundância garante que o atendimento não seja interrompido. Todo o ambiente foi estruturado para facilitar a instalação e a manutenção, e os logs das aplicações ficam separados, o que simplifica a identificação de problemas..** 

🔧 PARTE 1 — Comandos Revisados e Corrigidos \
# 1. Gerenciar o Ambiente

|          Comando 	                |         Explicação                                                   |
|-----------------------------------|----------------------------------------------------------------------|
|docker compose up -d --build   	|  ✅ Constrói imagens e sobe TUDO em segundo plano                   |
|docker compose up -d	            |  ✅ Sobe os contêineres (sem reconstruir)                           |                     
|docker compose down	            |  ✅ Para e remove contêineres e rede → não apaga imagens nem dados   |
|docker compose down --rmi all      |  ⚠️ Para tudo e apaga as imagens → usar só quando necessário limpar tudo |
|docker compose restart	            |     Reinicia todos os contêineres sem recriar                         |
|docker compose restart tomcat-a	|     Reinicia apenas o tomcat-a                                        |
|docker compose stop	            |     Para tudo sem remover                                             |
|docker compose start	            |     Liga tudo que estava parado                                       |

# 2. Verificação e Status

|       Comando	                 |          Explicação                               |
|--------------------------------|---------------------------------------------------|
|docker compose ps	          |  ✅ Lista contêineres ativos|
|docker compose ps -a	       | ✅ Lista TODOS (ativos e parados) |
|docker compose logs -f	       | ✅ Acompanha logs dos dois em tempo real |
|docker compose logs -f tomcat-a	|✅ Acompanha só o tomcat-a |
|docker compose logs -f tomcat-b	|✅ Acompanha só o tomcat-b |
|docker compose logs --tail=50   |   tomcat-a	Mostra últimas 50 linhas do log |
|docker stats	                |    Mostra uso de CPU, memória e rede em tempo real|
|docker inspect tomcat-tomcat-a-1 |	Detalhes completos do contêiner |

# 3. Atualização de Aplicação — Sem Parar o Serviço ⭐

# ✅ ATUALIZAR TOMCAT-B PRIMEIRO
- cp novo-arquivo.war /opt/tomcat/apps-b/nome-aplicacao.war

# Aguardar ~30s e testar: http://localhost:8082/nome-aplicacao

# ✅ Tudo certo? Agora atualizar o tomcat-a
- cp novo-arquivo.war /opt/tomcat/apps-a/nome-aplicacao.war

# Aguardar e testar: http://localhost:8081/nome-aplicacao
⚠️ NÃO precisa reiniciar o contêiner! O Tomcat detecta o arquivo novo e recarrega sozinho. Se precisar forçar:

# Forçar recarrega de uma instância (não afeta a outra)
- docker compose restart tomcat-a

# 4. Editar Configurações do Banco (XML)
# Editar configuração
- vim /opt/tomcat/context-config/nome-aplicacao.xml

# ✅ Validar sintaxe do XML (não pode ter erro!)
- xmllint --noout /opt/tomcat/context-config/nome-aplicacao.xml

# Aplicar reiniciando uma instância por vez
- docker compose restart tomcat-b
# Testar...
- docker compose restart tomcat-a
5. Ver Logs Direto no Servidor

# Tomcat-A
- ls -lh /opt/tomcat/logs-a/
- tail -f /opt/tomcat/logs-a/catalina.out
- tail -n 100 /opt/tomcat/logs-a/catalina.out

# Tomcat-B
- ls -lh /opt/tomcat/logs-b/
- tail -f /opt/tomcat/logs-b/catalina.out
- tail -n 100 /opt/tomcat/logs-b/catalina.out

6. Comandos Adicionais Úteis

|     Comando   |	                         Explicação |
|------------------------|----------------------------------|
|docker compose build	| Reconstrói imagens sem reiniciar
|docker compose up -d --force-recreate tomcat-a |	Recria só o tomcat-a (mantém volumes) |
|docker compose pull |	Atualiza imagens base (quando houver) |
|docker system df	Mostra espaço em disco usado pelo Docker |
|docker system prune |	Limpa recursos não usados (cuidado!) |
|docker volume ls |	Lista volumes persistentes |
|docker compose exec tomcat-a bash |	🆗 Entra dentro do contêiner tomcat-a |
|docker compose exec tomcat-b ls -la /usr/local/tomcat/webapps/	| Executa comando dentro do contêiner sem entrar |

# � PARTE 2 — Adicionar Nova Aplicação
- Resposta direta: ✅ NÃO precisa mexer no Dockerfile! É só preparar os arquivos e pastas.
- Passo a Passo — Adicionar Nova App

1. Colocar o Arquivo .war nas Duas Pastas

- cd /opt/tomcat

# Copiar para as duas instâncias
- cp caminho/nova-aplicacao.war ./apps-a/
- cp caminho/nova-aplicacao.war ./apps-b/

# Confirmar
- ls -lh ./apps-a/nova-aplicacao.war
- ls -lh ./apps-b/nova-aplicacao.war
2. Criar o Arquivo de Contexto (XML)
bash
# Criar configuração de banco
vim ./context-config/nova-aplicacao.xml
Conteúdo modelo:
xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>

  <Resource name="jdbc/aghos"
            global="jdbc/aghos"
            auth="Container"
            type="javax.sql.DataSource"
            driverClassName="oracle.jdbc.driver.OracleDriver"
            url="jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=IPDOBANCO)(PORT=1521))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=PDBHS19)))"
            username="NOMEDOSCHEMA"
            password="SENHA"
            maxActive="100"
            maxIdle="20"
            minIdle="5"
            maxWait="10000"
            oracle.jdbc.mapDateToTimestamp="false"
  />

  <Resource name="jdbc/cnes"
            global="jdbc/cnes"
            auth="Container"
            type="javax.sql.DataSource"
            driverClassName="org.firebirdsql.jdbc.FBDriver"
            url="jdbc:firebirdsql://IPDOBANCO:3050//home/IMPORTACAO_CNES/NOMEDOSCEMA/CNES.GDB?charSet=UTF8"
            username="sysdba"
            password="masterkey"
            maxActive="100"
            maxIdle="20"
            minIdle="5"
            maxWait="10000"
  />

</Context>
3. Validar e Aplicar
bash
# Validar XML
xmllint --noout ./context-config/nova-aplicacao.xml

# Aguardar ~30s → Tomcat detecta e carrega sozinho!

# Verificar se carregou
docker compose logs tomcat-a | grep nova-aplicacao
docker compose logs tomcat-b | grep nova-aplicacao
4. Acessar
http://192.168.10.14:8081/nova-aplicacao ✅
http://192.168.10.14:8082/nova-aplicacao ✅
💡 Resumo: Nova aplicação = 1 arquivo .war × 2 pastas + 1 arquivo XML → pronto! Sem alterar compose nem Dockerfile!
📋 Resumo Rápido de Referência
Tabela
|Situação   |     	O que fazer     |
|------------|-----------------------|
|Subir tudo	 | docker compose up -d  |
|Parar tudo	 | docker compose down |
|Atualizar app |	Copiar .war para apps-b/ → testar → copiar para apps-a/ |
|Mudar config do banco |	Editar XML → reiniciar tomcat-b → testar → reiniciar tomcat-a |
|Nova aplicação |	.war em apps-a e apps-b + .xml em context-config |
|Ver logs |	docker compose logs -f ou tail -f /opt/tomcat/logs-a/catalina.out |
|Limpar tudo |	docker compose down --rmi all + docker compose up -d --build |
|Espaço em disco |	docker system df → docker system prune |



---


## 🔄 Sobre a Atualização — Só copiar o .war já basta?

### ✅ Resposta direta: **SIM! O Tomcat faz tudo sozinho!**

O Tomcat 7.0.47 tem um recurso chamado **auto-deploy** que já vem ativado por padrão. Funciona assim:

```
Você copia itssaude.com.br.war ← versão nova
         ↓
Tomcat detecta que o arquivo mudou
         ↓
Apaga a pasta antiga descompactada automaticamente
         ↓
Descompacta o WAR novo
         ↓
Sobe a aplicação nova
         ↓
Pronto! ✅
```

**Você NÃO precisa:**
- ❌ Apagar a pasta descompactada manualmente
- ❌ Editar nenhum arquivo de configuração
- ❌ Reiniciar o contêiner (na maioria das vezes)

O Tomcat **substitui tudo sozinho** quando percebe que o `.war` foi alterado. 🎉

---

## ⏰ Quando PRECISO forçar reinício?

Copiar o `.war` já basta em **90% dos casos**. Mas existem situações onde o Tomcat não percebe ou não consegue recarregar sozinho:

| Situação | Precisa reiniciar? | O que fazer |
|---|---|---|
| Apenas trocar o `.war` | Não ✅ | Só copiar e aguardar ~30s |
| Trocar o `.war` e a aplicação não carrega | Sim ⚠️ | `docker compose restart tomcat-a` |
| Alterou arquivo `.xml` de configuração do banco | Sim ✅ | `docker compose restart tomcat-b` → testar → `tomcat-a` |
| Alterou variáveis de ambiente (`JAVA_OPTS` etc) | Sim ✅ | `docker compose up -d --force-recreate tomcat-a` |
| Aplicação travou ou não responde | Sim ✅ | `docker compose restart tomcat-b` |
| Quer garantir reinício limpo | Opcional 💡 | `docker compose restart tomcat-a` |

---

## 📌 Resumo Prático para o Dia a Dia

### Fluxo Padrão (só copiar):
```bash
# 1. Copia novo WAR para B primeiro
cp nova.war ./apps-b/nome.war
# ⏳ Aguardar ~30s e testar porta 8082

# 2. Tudo certo? Copia para A
cp nova.war ./apps-a/nome.war
# ⏳ Aguardar ~30s e testar porta 8081
```
> ✅ Pronto! Sem parada, sem reinício manual, sem apagar pastas!

### Fluxo com Reinício (quando necessário):
```bash
# 1. Copia para B
cp nova.war ./apps-b/nome.war
# 2. Força reinício do B
docker compose restart tomcat-b
# ⏳ Testar porta 8082

# 3. Tudo certo? Faz o mesmo no A
cp nova.war ./apps-a/nome.war
docker compose restart tomcat-a
```

### Por que fazer B primeiro?
Assim a instância **A continua atendendo** enquanto B reinicia, e vice-versa — **ninguém fica sem serviço!** 🎯

---

## 💡 Dica Extra — Confirmar se Atualizou

Para verificar se a versão nova já está no ar:
```bash
# Ver nos logs se carregou
docker compose logs tomcat-b | grep "Deployment"

# Ou ver diretamente se a pasta foi recriada
ls -lt /opt/tomcat/apps-b/
# ← a pasta da aplicação deve aparecer com horário atual
```

---

Ficou claro agora? Resumindo:
- ✅ Só copiar o `.war` → Tomcat faz o resto sozinho
- ✅ Não precisa apagar pasta descompactada
- ⚠️ Reinício só quando mudar XML, variáveis ou a app não recarregar sozinha
- 🎯 Sempre atualiza **B primeiro**, depois **A** → serviço nunca para! 😊

Mais alguma dúvida? Estou à disposição! 💪