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

    public partial class Wisielecc : Form, IObserver
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

        private WrongGuesses wrongGuesses = new WrongGuesses();

        public Wisielecc()
        {
            InitializeComponent();
        }

        private void quessClick(object sender, EventArgs e)
        {
            Button wybor = sender as Button;
            wybor.Enabled = false; // sprawia ze po użyciu jest nieaktywne

            if (WordsManager.getInstance().checkWord(wybor.Text))
            {
                tesktowe1.Text = " Wygrałeś !!! :) ";

                bool result = (MessageBox.Show("Czy chcesz zacząć od nowa?", "Wygrałeś !!!",
   MessageBoxButtons.YesNo, MessageBoxIcon.Question, MessageBoxDefaultButton.Button2)
   == DialogResult.Yes);
                newGame(result);
            }
            else
            {
                if (wrongGuesses.getCounter() >= Images.Length - 1)
                {
                    tesktowe1.Text = " Przegrałeś !!! :( ";

                    bool result = (MessageBox.Show("Czy chcesz zacząć od nowa?", "Przegrałeś",
           MessageBoxButtons.YesNo, MessageBoxIcon.Question, MessageBoxDefaultButton.Button2)
           == DialogResult.Yes);
                    newGame(result);
                }
            }
        }


        private void Wisielecc_Load(object sender, EventArgs e)
        {
            WordsManager.getInstance().addObserver(wrongGuesses);
            WordsManager.getInstance().addObserver(this);

            WordsManager.getInstance().loadWords(File.ReadAllLines("slowa.txt"));
            WordsManager.getInstance().randomWord();
        }

        private void button1_Click(object sender, EventArgs e)
        {
            MessageBoxButtons buttons = MessageBoxButtons.OK;
            MessageBox.Show(WordsManager.getInstance().getSecretWord(), "Słowo:", buttons);
        }

        private void toolStripMenuItem2_Click_1(object sender, EventArgs e)
        {
            MessageBoxButtons buttons = MessageBoxButtons.OK;
            MessageBox.Show("Dominik Pyrda a.k.a Loczek \n Marcin Srebro a.k.a Albatros", "Twórcy", buttons);
        }

        private void menuStrip1_ItemClicked(object sender, ToolStripItemClickedEventArgs e)
        {

        }

        private void toolStripMenuItem6_Click(object sender, EventArgs e)
        {
            bool result = (MessageBox.Show("Czy na pewno chcesz zakończyć?", "Exit",
                  MessageBoxButtons.YesNo, MessageBoxIcon.Question, MessageBoxDefaultButton.Button2)
                  == DialogResult.Yes);
            newGame(!result);
        }

        private void toolStripMenuItem5_Click(object sender, EventArgs e)
        {
            bool result = (MessageBox.Show("Czy na pewno chcesz zacząć od nowa?", "Nowa Gra",
                  MessageBoxButtons.YesNo, MessageBoxIcon.Question, MessageBoxDefaultButton.Button2)
                  == DialogResult.Yes);
            if (result)
            {
                newGame(result);
            }
        }

        private void toolStripComboBox1_Click(object sender, EventArgs e)
        {

        }

        private void czarnyToolStripMenuItem_Click(object sender, EventArgs e)
        {
            this.BackColor = System.Drawing.Color.Black;
        }

        private void czerwonyToolStripMenuItem_Click(object sender, EventArgs e)
        {
            this.BackColor = System.Drawing.Color.Red;
        }

        private void zielonyToolStripMenuItem_Click(object sender, EventArgs e)
        {
            this.BackColor = System.Drawing.Color.Green;
        }

        private void niebieskiToolStripMenuItem_Click(object sender, EventArgs e)
        {
            this.BackColor = System.Drawing.Color.Blue;
        }

        private void czarnyToolStripMenuItem1_Click(object sender, EventArgs e)
        {
            this.ForeColor = System.Drawing.Color.Black;
        }

        private void czerwonyToolStripMenuItem1_Click(object sender, EventArgs e)
        {
            this.ForeColor = System.Drawing.Color.Red;
        }

        private void zielonyToolStripMenuItem1_Click(object sender, EventArgs e)
        {
            this.ForeColor = System.Drawing.Color.Green;
        }

        private void niebieskiToolStripMenuItem1_Click(object sender, EventArgs e)
        {
            this.ForeColor = System.Drawing.Color.Blue;
        }

        public void update(string value)
        {
            if (value != null)
            {
                lbltekst.Text = "";
                for (int index = 0; index < value.Length; index++)
                {
                    lbltekst.Text += value.Substring(index, 1);
                    lbltekst.Text += " ";
                }
            }
            else
            {
                if (wrongGuesses.getCounter() < Images.Length)
                {
                    wisielecImage.Image = Images[wrongGuesses.getCounter()];
                }
            }
        }

        private void newGame(bool newGame)
        {
            if (newGame)
            {
                wrongGuesses.reset();
                wisielecImage.Image = Images[wrongGuesses.getCounter()];
                WordsManager.getInstance().randomWord();
                tesktowe1.Text = "";
                for (int i = 0; i < this.Controls.Count; i++)
                {
                    this.Controls[i].Enabled = true;
                }
            }
            else
            {
                Application.Exit();
            }
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
        private static WordsManager instance = null;

        private static char[] SEPARATOR = { ',' };

        private string[] words;

        private string secretWord;

        private string guessedWord;

        private List<IObserver> observers = new List<IObserver>();

        public WordsManager()
        {

        }

        /// <summary>
        /// Singleton - wzorzec projektowy
        /// http://www.algorytm.org/wzorce-projektowe/singleton-singleton.html
        /// </summary>
        /// <returns>WordsManager</returns>
        public static WordsManager getInstance()
        {
            if (instance == null)
            {
                instance = new WordsManager();
            }
            return instance;
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

            notifyObservers(guessedWord);
        }

        public string getSecretWord()
        {
            return secretWord;
        }

        public bool checkWord(string value)
        {
            if (secretWord.Contains(value))
            {
                char[] temp = guessedWord.ToCharArray();
                char[] find = secretWord.ToCharArray();
                char questChar = value.ElementAt(0);

                for (int index = 0; index < find.Length; index++)
                {
                    if (find[index] == questChar)
                    {
                        temp[index] = questChar;
                    }

                }
                guessedWord = new string(temp);
                notifyObservers(guessedWord);
            }
            else
            {
                notifyObservers(null);
            }

            return secretWord.Equals(guessedWord);
        }

        private void notifyObservers(string value)
        {
            foreach(IObserver o in observers)
            {
                o.update(value);
            }
        }

        public void addObserver(IObserver observer)
        {
            this.observers.Add(observer);
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
    public class WrongGuesses : IObserver
    {
        private int counter;

        public WrongGuesses()
        {
            reset();
        }

        public int getCounter()
        {
            return counter;
        }

        public void reset()
        {
            counter = 0;
        }

        public void update(string value)
        {
            if (value == null)
            {
                counter++;
            }
        }
    }
}
