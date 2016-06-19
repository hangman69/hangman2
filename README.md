using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.IO;
using Wisielec.Utils;

namespace Wisielec
{

    public partial class Wisielecc : Form
    {

        private Bitmap[] Images = { Wisielec.Properties.Resources._1, // obrazki
                                 Wisielec.Properties.Resources._2,
                                 Wisielec.Properties.Resources._3,
                                 Wisielec.Properties.Resources._4,
                                 Wisielec.Properties.Resources._5,
                                 Wisielec.Properties.Resources._6,
                                 Wisielec.Properties.Resources._7,
                                 Wisielec.Properties.Resources._8,
                                 Wisielec.Properties.Resources._9 };

        

        public Wisielecc()
        {
            InitializeComponent();
        }
      
        private void newGame(bool newGame)
        {
           
        }
    }
}

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Wisielec.Utils
{
    public class WordsManager
    {

        private static char[] SEPARATOR = { ',' };

        private string[] words;

        private string secretWord;

        private string guessedWord;

        public WordsManager()
        {

        }

        public void loadWords(string[] fileLines)
        {
            words = new string[fileLines.Length];  //robimy dynamicznie tablice o dlugosci jaka ma readtext
            int index = 0;
            foreach (string s in fileLines)
            {
                string[] line = s.Split(SEPARATOR);
                words[index++] = line[1];
            }
        }

        public string[] getWords()
        {
            return words;
        }

        public void randomWord()
        {
            int quessIndex = (new Random()).Next(words.Length);
            secretWord = words[quessIndex];

            guessedWord = "";
            for (int index = 0; index < secretWord.Length; index++)
            {
                guessedWord += "_";
            }

           
        }

        
        

    
    }
}
