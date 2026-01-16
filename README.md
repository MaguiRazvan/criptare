using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

class SubstitutionCipher
{
    private const string ALPHABET = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    private const string STD_FREQUENCY_ENG = "ETAOINSHRDLCUMWFGYPBVKJXQZ";

    public static string GenerateRandomKey()
    {
        Random rnd = new Random();
        return new string(ALPHABET.OrderBy(x => rnd.Next()).ToArray());
    }

    public static string Encrypt(string plaintext, string key)
    {
        return Transform(plaintext, ALPHABET, key);
    }

    public static string Decrypt(string ciphertext, string key)
    {
        return Transform(ciphertext, key, ALPHABET);
    }

    private static string Transform(string text, string sourceAlpha, string targetAlpha)
    {
        StringBuilder result = new StringBuilder();
        foreach (char c in text)
        {
            char upperC = char.ToUpper(c);
            int index = sourceAlpha.IndexOf(upperC);

            if (index != -1)
            {
                char newChar = targetAlpha[index];
                result.Append(char.IsLower(c) ? char.ToLower(newChar) : newChar);
            }
            else
            {
                result.Append(c);
            }
        }
        return result.ToString();
    }

    public static void Cryptanalysis(string ciphertext)
    {
        var charCounts = ciphertext
            .Where(char.IsLetter)
            .GroupBy(c => char.ToUpper(c))
            .Select(group => new { Letter = group.Key, Count = group.Count() })
            .OrderByDescending(x => x.Count)
            .ToList();

        string cipherFreqOrder = "";
        foreach (var item in charCounts)
        {
            cipherFreqOrder += item.Letter;
        }

        char[] guessedKeyArray = new char[26];
        for (int i = 0; i < 26; i++) guessedKeyArray[i] = '?';

        for (int i = 0; i < cipherFreqOrder.Length && i < STD_FREQUENCY_ENG.Length; i++)
        {
            char cipherChar = cipherFreqOrder[i];
            char plainChar = STD_FREQUENCY_ENG[i];
            
            int indexInAlphabet = ALPHABET.IndexOf(cipherChar);
            if (indexInAlphabet != -1)
            {
                guessedKeyArray[indexInAlphabet] = plainChar;
            }
        }

        StringBuilder attempt = new StringBuilder();
        foreach (char c in ciphertext)
        {
            if (char.IsLetter(c))
            {
                int index = ALPHABET.IndexOf(char.ToUpper(c));
                attempt.Append(guessedKeyArray[index]);
            }
            else
            {
                attempt.Append(c);
            }
        }

        Console.WriteLine("\n--- Cryptanalysis Result ---");
        Console.WriteLine(attempt.ToString());
    }

    static void Main()
    {
        string key = GenerateRandomKey();
        Console.WriteLine($"Key: {key}");

        string plaintext = "THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG. CRYPTOGRAPHY IS INTERESTING.";
        Console.WriteLine($"Original: {plaintext}");

        string encrypted = Encrypt(plaintext, key);
        Console.WriteLine($"Encrypted: {encrypted}");

        string decrypted = Decrypt(encrypted, key);
        Console.WriteLine($"Decrypted: {decrypted}");

        Cryptanalysis(encrypted);

        Console.ReadKey();
    }
}
