# Instruções de Build - QZ Tray

## Pré-requisitos

https://download.bell-sw.com/java/11.0.27+9/bellsoft-jdk11.0.27+9-windows-amd64.zip

### Ferramentas Necessárias

1. **Java Development Kit (JDK) 17+**
   - Necessário para compilar o código-fonte
   - Baixar: [Oracle JDK](https://www.oracle.com/java/technologies/downloads/) ou [OpenJDK](https://openjdk.org/)

2. **Apache Ant 1.10+**
   - Sistema de build do projeto
   - Instalar via Chocolatey (requer privilégios de administrador):
     ```powershell
     choco install ant
     ```
   - Ou baixar manualmente: [Apache Ant](https://ant.apache.org/bindownload.cgi)

3. **NSIS (Nullsoft Scriptable Install System)**
   - Para criar o instalador Windows
   - Instalar via Chocolatey:
     ```powershell
     choco install nsis
     ```
   - Ou baixar: [NSIS Download](https://nsis.sourceforge.io/Download)

4. **BellSoft JDK 11 (para bundling)**
   - Java runtime que será incluído no instalador
   - Baixar: [BellSoft Liberica JDK 11](https://bell-sw.com/pages/downloads/#jdk-11-lts)
   - **Versão necessária**: `bellsoft-jdk11.0.27+9-windows-amd64.zip`

## Configuração Inicial

### 1. Baixar e Preparar o JRE

```powershell
# Baixe o arquivo bellsoft-jdk11.0.27+9-windows-amd64.zip
# Coloque o arquivo na raiz do repositório (D:\Projetos\tray\)
```

### 2. Configurar Propriedades do Build

Crie/edite o arquivo `ant/private/private.properties`:

```properties
# Configuração do JRE bundled
jlink.java.target=D:/Projetos/tray/out/jlink/jdk-bellsoft-x86_64-windows-11.0.27_9

# Certificado auto-assinado (desenvolvimento)
# Para produção, adicione certificados válidos aqui
# keystore=caminho/para/certificado.p12
# keypass=senha_do_certificado
# tsaurl=http://timestamp.server.com
```

## Processo de Build

### Build Completo (JAR + Instalador Windows)

```powershell
# 1. Navegar até o diretório do projeto
cd D:\Projetos\tray

# 2. Extrair o JRE (se ainda não estiver extraído)
mkdir -Force "out\jlink"
Expand-Archive -Path "bellsoft-jdk11.0.27+9-windows-amd64.zip" -DestinationPath "out\jlink\jdk-bellsoft-x86_64-windows-11.0.27_9" -Force
Move-Item -Path "out\jlink\jdk-bellsoft-x86_64-windows-11.0.27_9\jdk-11.0.27\*" -Destination "out\jlink\jdk-bellsoft-x86_64-windows-11.0.27_9\" -Force
Remove-Item "out\jlink\jdk-bellsoft-x86_64-windows-11.0.27_9\jdk-11.0.27" -Force

# 3. Executar build do instalador
ant nsis
```

### Build Apenas do JAR

```powershell
# Se você só precisa do arquivo JAR sem o instalador
ant distribute
```

### Limpar Build Anterior

```powershell
# ATENÇÃO: Isso remove o diretório out/, incluindo o JRE extraído
ant clean
# Você precisará re-extrair o JRE depois disso
```

## Estrutura de Saída

Após o build bem-sucedido, os arquivos estarão em:

```
out/
├── dist/
│   ├── qz-tray.jar              # Aplicação principal (assinada)
│   ├── qz-tray.exe              # Launcher Windows (assinado)
│   ├── qz-tray-console.exe      # Launcher com console para debug (assinado)
│   ├── uninstall.exe            # Desinstalador (assinado)
│   ├── libs/                    # Bibliotecas nativas (DLLs assinadas)
│   ├── runtime/                 # Java runtime bundled
│   └── demo/                    # Arquivos de demonstração
└── qz-tray-2.2.6-SNAPSHOT-x86_64.exe  # INSTALADOR FINAL (~102MB)
```

## Problemas Comuns e Soluções

### Erro: SSL Certificate (PKIX path building failed)

**Problema**: Firewall corporativo bloqueia downloads de dependências.

**Solução**: 
- Baixe manualmente o JRE do BellSoft
- Coloque o ZIP na raiz do projeto
- Configure `jlink.java.target` em `ant/private/private.properties`

### Erro: JRE não encontrado após `ant clean`

**Problema**: `ant clean` remove o diretório `out/` incluindo o JRE extraído.

**Solução**: 
- Re-extrair o JRE antes de cada build completo (veja comandos acima)
- Ou modifique `build.xml` para preservar `out/jlink/`

### Erro: "jdeps.exe not found" ou "release file not found"

**Problema**: JRE não extraído corretamente ou estrutura de diretórios errada.

**Solução**:
```powershell
# Verificar estrutura correta:
ls "out\jlink\jdk-bellsoft-x86_64-windows-11.0.27_9"
# Deve conter: bin/, jmods/, lib/, conf/, release
```

### Avisos sobre Certificado Auto-Assinado

**Aviso**: `The signer's certificate is self-signed`

**Explicação**: Normal em ambiente de desenvolvimento. Para produção, use certificado de CA válida.

### Instalação Falha em Alguns Computadores

**Problema**: "installation failed during preinstall step"

**Solução Aplicada**: Modificamos `ant/windows/windows-installer.nsi.in` para usar `Exec` ao invés de `ExecWait`, permitindo que a instalação continue mesmo se o preinstall falhar.

## Testes Recomendados

Antes de distribuir o instalador:

1. **Teste em máquina limpa** (sem Java instalado)
   - Verificar se a aplicação inicia sem pedir Java
   - Confirmar que o runtime bundled está funcionando

2. **Teste a instalação**
   - Executar `qz-tray-2.2.6-SNAPSHOT-x86_64.exe`
   - Verificar se instala em `C:\Program Files\QZ Tray`
   - Confirmar atalhos criados

3. **Teste a desinstalação**
   - Usar o desinstalador ou Painel de Controle
   - Verificar se remove todos os arquivos

4. **Teste funcionalidade**
   - Iniciar aplicação
   - Testar impressão
   - Verificar comunicação WebSocket

## Notas de Versão

- **Versão Atual**: 2.2.6-SNAPSHOT
- **Java Runtime Bundled**: BellSoft JDK 11.0.27+9
- **JavaFX**: 19 monocle (incluído)
- **Tamanho do Instalador**: ~102MB (com JRE)
- **Plataforma**: Windows x86_64

## Referências

- [Repositório Original QZ Tray](https://github.com/qzind/tray)
- [Documentação Apache Ant](https://ant.apache.org/manual/)
- [NSIS Documentation](https://nsis.sourceforge.io/Docs/)
- [BellSoft Liberica JDK](https://bell-sw.com/liberica/)

---

**Última atualização**: 4 de dezembro de 2025
