# 🤖 C# AI Agent — Microsoft Agent Framework

Bu proje, **Microsoft Agent Framework** ve **GitHub Models** kullanarak C# ile yazılmış basit bir AI agent örneğidir. Agent, kullanıcıdan gelen soruları yanıtlar ve gerektiğinde tanımlı C# metodlarını (tool) otomatik olarak çağırır.

## 🚀 Özellikler

- Tool-calling desteği (`AIFunctionFactory`)
- GitHub Models üzerinden ücretsiz GPT-4.1-mini kullanımı
- `.NET User Secrets` ile güvenli token yönetimi
- `Microsoft.Extensions.AI` soyutlama katmanı sayesinde provider bağımsız mimari

## 📦 Kullanılan Paketler

| Paket | Versiyon |
|-------|----------|
| Microsoft.Agents.AI | 1.10.0 |
| Microsoft.Extensions.AI | latest |
| Microsoft.Extensions.AI.OpenAI | latest |
| Azure.AI.OpenAI | 2.1.0 |

## ⚙️ Kurulum

### 1. Repoyu klonla
```bash
git clone https://github.com/kullanici-adin/repo-adin.git
cd repo-adin
```

### 2. GitHub token oluştur

1. [github.com/settings/tokens](https://github.com/settings/tokens) adresine git
2. **"Generate new token (classic)"** butonuna tıkla
3. Token'a bir isim ver (örn: `ai-agent-token`)
4. **Expiration** seç (örn: 30 days)
5. İzinler kısmında **`models:read`** kutucuğunu işaretle
6. **"Generate token"** butonuna tıkla
7. Çıkan token'ı kopyala — bir daha göremezsin!

> ⚠️ Token'ı kod içine veya Git'e asla yazma.

### 3. Token'ı User Secrets ile kaydet
```bash
dotnet user-secrets init
dotnet user-secrets set "GitHubToken" "github_pat_..."
```

### 4. Çalıştır
```bash
dotnet run
```

## 📝 Kod Örneği

```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;
using OpenAI.Chat;
using System.ComponentModel;

var config = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var githubToken = config["GitHubToken"];
var endpoint = new Uri("https://models.github.ai/inference");
var model = "openai/gpt-4.1-mini";

[Description("Verilen müşteri için bu günkü sipariş sayısını döndür")]
static int GetOrderCountToday(
    [Description("Müşteri adı")] string customerName)
{
    Console.WriteLine("Tool Working...");
    return 12;
}

var openAIClientOptions = new OpenAIClientOptions { Endpoint = endpoint };

AIAgent aIAgent = new ChatClient(model, new ApiKeyCredential(githubToken), openAIClientOptions)
    .AsIChatClient()
    .AsAIAgent(
        instructions: "Sen yardımcı bir asistansın, kısa cevaplar ver",
        name: "Test Agent",
        tools: [AIFunctionFactory.Create(GetOrderCountToday)]);

Console.Write("Mesajınız: ");
string msg = Console.ReadLine()!;
var response = await aIAgent.RunAsync(msg);
Console.WriteLine(response);
```

## 🎬 Demo

![Agent çalışma ekranı](demo.png)

Agent'ın tool-calling özelliğini gösteren terminal çıktısı:
- `2+2` sorusuna matematik cevabı
- `X isimli müşterinin sipariş sayısı` sorusunda **Tool Working...** çıktısıyla `GetOrderCountToday` metodunu otomatik çağırması
- Bilgisi dışındaki sorularda dürüstçe "bilmiyorum" demesi

## 🔍 Öğrendiklerim

- `OpenAI.Chat.ChatClient` doğrudan `AsAIAgent()` kabul etmiyor — araya `.AsIChatClient()` koymak gerekiyor
- GitHub Models'da model adı `"gpt-4.1-mini"` değil, `"openai/gpt-4.1-mini"` olmalı
- GitHub token'ında `models:read` izni olmazsa HTTP 403 hatası alırsın
- Token'ı asla kod içine yazma — `.NET User Secrets` kullan

## 📄 Lisans

MIT
