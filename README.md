using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using HtmlAgilityPack;
using Newtonsoft.Json;

class Program
{
    static void Main(string[] args)
    {

    //adicione o caminho 
        string directoryPath = "C:/Caminho/Para/";
        string fileName = "resultados.json";
        string filePath = Path.Combine(directoryPath, fileName);

        List<string> palavrasChave = new List<string>();

        if (File.Exists(filePath))
        {
            palavrasChave = CarregarPalavrasChave(filePath);
        }

        while (true)
        {
            Console.WriteLine("Deseja adicionar novas palavras-chave? (S/N)");
            string resposta = Console.ReadLine();
            if (resposta.ToLower() == "s")
            {
                Console.WriteLine("Insira as palavras-chave separadas por vírgula:");
                string palavrasChaveInput = Console.ReadLine();
                palavrasChave.AddRange(palavrasChaveInput.Split(','));

                SalvarPalavrasChave(filePath, palavrasChave);
            }

            Dictionary<string, string> resultados = new Dictionary<string, string>();
            foreach (string palavraChave in palavrasChave)
            {
                string resultado = RealizarPesquisa(palavraChave);
                resultados.Add(palavraChave, resultado);
            }

            string data = DateTime.Now.ToString("yyyy-MM-dd");
            string diaSemana = DateTime.Now.DayOfWeek.ToString();

            SalvarResultados(filePath, data, diaSemana, resultados);

            Console.WriteLine("Resultados salvos com sucesso.");

            Console.WriteLine("Pressione qualquer tecla para continuar...");
            Console.ReadKey();
        }
    }

    static List<string> CarregarPalavrasChave(string filePath)
    {
        string json = File.ReadAllText(filePath);
        dynamic data = JsonConvert.DeserializeObject(json);
        List<string> palavrasChave = new List<string>();
        foreach (var resultado in data["Resultados"])
        {
            palavrasChave.Add(resultado.ToString());
        }
        return palavrasChave;
    }

    static void SalvarPalavrasChave(string filePath, List<string> palavrasChave)
    {
        var dataResults = new
        {
            Resultados = palavrasChave
        };

        string json = JsonConvert.SerializeObject(dataResults, Formatting.Indented);
        File.WriteAllText(filePath, json);
    }

    static string RealizarPesquisa(string palavraChave)
    {
        string url = $"https://www.google.com/search?q={palavraChave}";

        string html;
        using (WebClient client = new WebClient())
        {
            html = client.DownloadString(url);
        }

        HtmlDocument doc = new HtmlDocument();
        doc.LoadHtml(html);
        HtmlNode primeiroResultado = doc.DocumentNode.SelectSingleNode("//div[@class='BNeawe vvjwJb AP7Wnd']");

        return primeiroResultado.InnerText;
    }

    static void SalvarResultados(string filePath, string data, string diaSemana, Dictionary<string, string> resultados)
    {
        var dataResults = new
        {
            Data = data,
            DiaSemana = diaSemana,
            Resultados = resultados
        };

        string json = JsonConvert.SerializeObject(dataResults, Formatting.Indented);
        File.WriteAllText(filePath, json);
    }
}
