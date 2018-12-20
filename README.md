# Kiosknumarator
Kiosk Numaratör Programı C#

C# ile geliştirilmiş basit bir arayüze sahip, termal yazıcıdan günlük sıra numarası veren program.



    public partial class Form1 : Form
    {
        // PrintDocument yazdir = new PrintDocument();
        const string txt_SiraAliniyor = "Sıra Alınıyor...";
        const string txt_SiraAl = "Sıra Al";

        int eni = 600;
        int boyu = 600;
        int sayac = 0;
        //int sure = 0;
        string NumaraKlasoru = null;
        string NumaraDosyasi = null;
        //string UpdateDosyası = null;

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            printDocument1.DefaultPageSettings.PaperSize = new PaperSize("Barkod", eni, boyu);
            printDocument1.PrinterSettings.PrinterName = "Brother HL-1110 series";
            //yazdir.PrinterSettings.PrinterName = "Barkod";

            timer1.Interval = 3000; // 2 sn
            NumaraKlasoru = System.IO.Path.GetDirectoryName(Application.ExecutablePath.Replace("bin\\Debug", "").Replace("bin\\Release", "")) + @"\Data\";
            NumaraDosyasi = NumaraKlasoru + "numaralar.txt";
        }

        private void button1_Click(object sender, EventArgs e)
        {
            try
            {
                button1.Text = txt_SiraAliniyor;
                button1.Enabled = false;

                // Klasör kontrol
                if (!Directory.Exists(NumaraKlasoru))
                {
                    try
                    {
                        Directory.CreateDirectory(NumaraKlasoru);
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show(this, NumaraKlasoru + " yaratılamadı! Detaylar : " + ex.Message, "HATA", MessageBoxButtons.OK, MessageBoxIcon.Error);
                        this.Close();
                        return;
                    }
                }

                // Txtkontrol
                string veriler = null;

                try
                {
                    if (!File.Exists(NumaraDosyasi))
                    {
                        veriler = string.Format("0;{0}", DateTime.Now.ToShortDateString());
                        File.WriteAllText(NumaraDosyasi, veriler);
                    }
                    else
                    {
                        veriler = File.ReadAllText(NumaraDosyasi);
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show(this, NumaraDosyasi + " yaratılamadı! Detaylar : " + ex.Message, "HATA", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    this.Close();
                    return;
                }

                if (string.IsNullOrEmpty(veriler))
                {
                    MessageBox.Show(this, "Hiç Veri Yok!", "HATA", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    this.Close();
                    return;
                }

                this.sayac = 0;
                string okunanTarih = null;

                try
                {
                    string[] temp = veriler.Split(';');
                    sayac = int.Parse(temp[0]);
                    okunanTarih = temp[1];
                }
                catch (Exception ex)
                {
                    MessageBox.Show(this, "Convert hatası! Detaylar : " + ex.Message, "HATA", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    this.Close();
                    return;
                }

                string suan = DateTime.Now.ToShortDateString();
                if (!okunanTarih.Equals(suan, StringComparison.Ordinal))
                {
                    sayac = 1;
                    veriler = string.Format("{0};{1}", sayac, suan);
                    File.WriteAllText(NumaraDosyasi, veriler);
                }
                else
                {
                    sayac++;
                    veriler = string.Format("{0};{1}", sayac, suan);
                    File.WriteAllText(NumaraDosyasi, veriler);
                }

                try
                {
                    printDocument1.Print();
                }
                catch
                {
                    MessageBox.Show("Yazcı ile ilgili bir sorun oluştu. Yazıcının açık olup olmadığını kontrol ediniz.", "Yazıcı Hatası");
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Bilinmeyen Bir hata oluştu! " + ex.Message, "Global Hata");
            }
            finally
            {
                timer1.Start();
            }
        }

        private void printDocument1_PrintPage(object sender, PrintPageEventArgs e)
        {
            e.Graphics.DrawString(DateTime.Now.ToLongDateString(), new Font("Arial", 10), new SolidBrush(Color.Black), new RectangleF(5, 5, this.eni, this.boyu));
            e.Graphics.DrawString("Sıra Numaranız", new Font("Arial", 10), new SolidBrush(Color.Black), new RectangleF(20, 35, this.eni, this.boyu));
            e.Graphics.DrawString(this.sayac.ToString(), new Font("Arial", 25), new SolidBrush(Color.Black), new RectangleF(40, 55, this.eni, this.boyu));
        }

        private void timer1_Tick(object sender, EventArgs e)
        {
            timer1.Stop();
            button1.Text = txt_SiraAl;
            button1.Enabled = true;
        }
    }
