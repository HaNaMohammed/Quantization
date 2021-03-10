
package quantization;
import java.awt.Graphics;
import java.io.File;
import java.util.*;
import java.awt.image.BufferedImage;
import java.awt.image.Raster;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import javax.imageio.ImageIO;
import java.util.Collections;

public class Quantization {
    public static class NonUniformQ
    {
        ArrayList< ArrayList<Double>  > Data;
        ArrayList< Double   > Sub;
        ArrayList< ArrayList<Double>   > Avg;
    /******************functions********************/   
        public NonUniformQ()
        {
            Data   = new ArrayList <ArrayList<Double>> ();
            Sub  = new ArrayList <Double>  ();
            Avg  = new ArrayList <ArrayList<Double>>  ();
        }
    }
     public static class Table
     {
         ArrayList<Double> lower= new ArrayList<>();
         ArrayList<Double> higher=new ArrayList<>();
         ArrayList<Double> Q=new ArrayList<>();
         ArrayList<Double> Q_1=new ArrayList<>();
     }
    public static int Min(ArrayList<Double> value)
    {
        double min=value.get(0);
        int posision=0;
         for(int j=0;j<value.size();j++)
        {
             if(value.get(j)<min)         //Minimum Condition
             { min = value.get(j);
                posision=j;}
             else 
                 continue;
         }
         //System.out.println("quantization.Quantization.Min()" + posision);
         return posision ;
    }
    
    public static double GetAvg (ArrayList<Double> value)
    {
        float sum=0;
        for (int i=0 ; i<value.size() ; i++)
        {
            sum+=value.get(i);
        }
        //System.out.println("quantization.Quantization.GetAvg()" + sum/value.size() );
        return sum/value.size();
    }
    public static ArrayList<Double> readImage(String filePath) throws IOException
    {
        BufferedImage Image =null;
	  File originalImage= new File(filePath);
		Image=ImageIO.read(originalImage);
		BufferedImage gray=new BufferedImage(Image.getWidth(), Image.getHeight(), BufferedImage.TYPE_BYTE_GRAY);
		Graphics g=Image.createGraphics();
		g.drawImage(Image, 0, 0, null);
		g.dispose();
		int k=0;
		int [][]pixels=new int[Image.getWidth()][Image.getHeight()];
                //System.out.println("width: "+Image.getWidth()+"height: "+Image.getHeight());
//		int []arrFinal=new int[Image.getHeight()*Image.getWidth()];
                ArrayList<Double> arrFinal =new ArrayList<>();
		Raster raster=Image.getData();
		for(int i=0;i<Image.getWidth();i++){
		    for(int j=0;j<Image.getHeight();j++)
                        {
			  pixels[i][j]=raster.getSample(i, j, 0);
                          double d=pixels[i][j];
			  arrFinal.add(d);
			  k++;
			}
                }
                try    //Put Compressed In File "Compressed.Txt"
               {
                   FileWriter FOut =new FileWriter("ImageWidthHight.txt");
                   String getHeight=""+(Image.getHeight());
                   FOut.write(getHeight);
                   FOut.write("\n");
                   String getWidth=""+(Image.getWidth());
                   FOut.write(getWidth);                 
                   FOut.close();
               }
               catch (Exception ex)
               {
                   System.out.println("Error Message");
               }
               
//        for(int i=0 ; i<pixels.length ; i++)
//        {
//            for (int j=0 ; j<pixels[i].length ; j++)
//            {
//                System.out.print(pixels[i][j]+" ");
//            }
//            System.out.println();
//        }

        return arrFinal;
    }
         
    public static void writeImage(ArrayList<Double> pixels,String outputFilePath)
    {
        int height ;
        int width ;
        ArrayList<Integer> temp =new ArrayList<>();
                try    //read Compressed from File "ImageH_W.Txt"
                    {
                        String pointer="";
                        File fileName =new File("ImageWidthHight.txt");
                        FileReader fileReader =  new FileReader(fileName);
                        BufferedReader bufferedReader =  new BufferedReader(fileReader);
                        int t=0;
                        while((pointer = bufferedReader.readLine()) != null)
                        {  
                            int y = Integer.parseInt(pointer); 
                            temp.add(y);
                            //System.out.println("POinter "+pointer );
                        }
                    }
                catch (Exception ex)
                      {
                        System.out.println("*-*Error Message");
                      }
                height=temp.get(0);
                width =temp.get(1);
        double[][] PixelsImage=new double[width][height];
        //System.out.println("width: "+width+"height: "+height);
        /******************************   1D-->2D   *******************************/
        int counter=0;
        for(int i=0 ; i<width ;i++)
        {
            for(int j=0 ; j<height ; j++)
            {
                //System.out.println(pixels.get(counter));
                PixelsImage[i][j]=pixels.get(counter);
                counter++;
            }
        }
        File fileout=new File(outputFilePath);
        BufferedImage image2=new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB );

        for(int x=0;x<width ;x++)
        {
            for(int y=0;y<height;y++)
            {
               // image2.setRGB(x,y,(PixelsImage[y][x]<<16)|(PixelsImage[y][x]<<8)|(PixelsImage[y][x]));
                int a=255;
                int pix=(int) PixelsImage[x][y];
                int p=(a<<24)|(pix<<16)|(pix<<8)|pix;
                image2.setRGB(x, y, p);
            }
        }
        try
        {
            ImageIO.write(image2, "jpg", fileout);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
    }
     public static void Quantization(ArrayList<Double> val , int NumOfLevel)
     {
         Table table= new Table();
         ArrayList<NonUniformQ>Tree =new ArrayList<>();
         ArrayList<Double>avgSplit=new ArrayList<>();
         ArrayList<Double>avgSplitCheck=new ArrayList<>();
         ArrayList<Double>Subcompare=new ArrayList<>();
         NonUniformQ Temp= new NonUniformQ();
         Temp.Data.add(val);
         Temp.Sub.add(0.0);
         Tree.add(Temp);
         Temp=new NonUniformQ();
         NonUniformQ QTemp= new NonUniformQ(); // temp
         while (true)
         {
             //System.out.println("Tree.get(Tree.size()-1).Data.size()" + Tree.get(Tree.size()-1).Data.size());
             if (Tree.get(Tree.size()-1).Data.size()== NumOfLevel)
             {
                  ArrayList<Double> []  Tempo0 = (ArrayList<Double>[])new ArrayList[100];   // array of arraylist
                  for (int s=0 ; s<100 ; s++)
                  {
                      Tempo0[s] = new ArrayList<Double>();
                  }
                 /**************************************AVGcheck***********************************/
                 for(int i=0 ; i<Tree.get(Tree.size()-1).Data.size() ; i++)
                 {
                            double avg;
                            avg=GetAvg(Tree.get(Tree.size()-1).Data.get(i));
                            ArrayList<Double> a=new ArrayList<>();
                            a.add(avg);
                            Tree.get(Tree.size()-1).Avg.add(a);
                            avgSplitCheck.add(avg);
                     
                 }
                      Subcompare=new ArrayList<>();
                     for (int x=0 ; x<val.size() ; x++)
                     {
                        int position; 
                        double t;
                         //System.out.println("avgSplitCheck : "+ avgSplitCheck.size());
                        for(int y=0 ; y<avgSplitCheck.size() ; y++)
                        { 
                            
                            t=Math.pow((val.get(x)) - (avgSplitCheck.get(y)),2);
                            Subcompare.add(t);
                        }
                        position=Min(Subcompare);
                        Subcompare=new ArrayList<>();
                        
                        Tempo0[position].add(val.get(x));
                        //System.out.println("quantization.Quantization.Quantization()"+ Tempo[position].get(0) );  
                     }
                     for(int c=0 ; c< avgSplitCheck.size() ; c++)
                     {
                         QTemp.Data.add(Tempo0[c]);
                         
                     }
                     Tree.add(QTemp);
                /****************************************check for *******************************************/
                int flag=0;
                for (int i=0 ; i< avgSplitCheck.size() ;i++)
                {
                    for(int j=0 ; j<Tree.get(Tree.size()-1).Data.get(Tree.get(Tree.size()-1).Data.size()-1).size() ; j++)
                    {
                        //System.out.println("check " +Tree.get(Tree.size()-2).Data.get(Tree.get(Tree.size()-2).Data.size()-1).get(j)+":"+j);
                        //System.out.println("check " +Tree.get(Tree.size()-1).Data.get(Tree.get(Tree.size()-1).Data.size()-1).get(j)+":"+j);
                        if(Tree.get(Tree.size()-1).Data.get(Tree.get(Tree.size()-2).Data.size()-1).get(j)!=Tree.get(Tree.size()-1).Data.get(Tree.get(Tree.size()-1).Data.size()-1).get(j))
                        {
                            
                            flag=1;
                            break;
                        }
                        else
                            flag=0;
                    }
                }
                if(flag==0)
                {
                  /************************************create table***************************************/
                   
                    table.lower.add(0.0);
                     double AVR = 0;
                   for(int p=0 ; p<NumOfLevel-1 ; p++)   //must put "-1"
                   {
                       double k=p;
                       AVR=(avgSplitCheck.get(p)+avgSplitCheck.get(p+1))/2;
                       table.Q.add(k);
                       table.Q_1.add(avgSplitCheck.get(p));
                       table.higher.add(AVR);
                       table.lower.add(AVR);
                       
                   }
                   double k=NumOfLevel-1;
                   table.Q.add(k);
                   table.Q_1.add(avgSplitCheck.get(avgSplitCheck.size()-1));
                   table.lower.add(AVR);
                   double max =Collections.max(val);
                   table.higher.add(max+1);  
                }

                   
                   /****************************print table*********************************************/
                   System.out.println("*********table*********");
                   System.out.println("Q  lower  higher  Q_1");
                   for(int q=0 ; q<table.Q_1.size() ; q++)
                   {
                       System.out.println((table.Q.get(q))+"  "+(table.lower.get(q))+"   " + (table.higher.get(q) )+"   "+ (table.Q_1.get(q)));
                       
                   }
                   break;
             }
             else 
             {
                 String TempData;
                 double TempSub;
                 double avg;
                 int counter =1;
                 NonUniformQ Q= new NonUniformQ(); // temp
               
                 int TreeSize = Tree.size()-1;
                 int DataSize = Tree.get(TreeSize).Data.size()-1;
                 for(int i=0 ; i<Tree.size() ; i++)
                 {
                  ArrayList<Double> []  Tempo = (ArrayList<Double>[])new ArrayList[100];   // array of arraylist
                  for (int s=0 ; s<100 ; s++)
                  {
                      Tempo[s] = new ArrayList<Double>();
                  }
                     //System.out.println("Tree.get(Tree.size()-1).Data.size()" + Tree.get(Tree.size()-1).Data.size() + "number of level "+ NumOfLevel);
                     if (Tree.get(Tree.size()-1).Data.size()== NumOfLevel)
                        {
                            break;
                        }
                      //System.out.println("Tree.size()" +Tree.size() );
                     for(int j=0 ; j<Tree.get(i).Data.size() ; j++)
                     {
                         //System.out.println("Tree.get(i).Data.size()" +Tree.get(i).Data.size() );
                         //System.out.println("Tree.get(i).Data.get(j).size()" +Tree.get(i).Data.get(j).size() );
                            double val1 ,val2;     // avg-1  && avg+1
                            avg=GetAvg(Tree.get(i).Data.get(j));
                            ArrayList<Double> a=new ArrayList<>();
                            a.add(avg);
                            Tree.get(i).Avg.add(a);
                            val1=avg-1;
                            val2=avg+1;
                            avgSplit.add(val1);
                            avgSplit.add(val2);
                    
                         
                     
                     }
                     Subcompare=new ArrayList<>();
                     for (int x=0 ; x<val.size() ; x++)
                     {
                        int position; 
                        double t;
                        for(int y=0 ; y<avgSplit.size() ; y++)
                        { 
                            
                            t=Math.pow((val.get(x)) - (avgSplit.get(y)),2);
                            Subcompare.add(t);
                        }
                        position=Min(Subcompare);
                        Subcompare=new ArrayList<>();
                        
                        Tempo[position].add(val.get(x));
                        //System.out.println("quantization.Quantization.Quantization()"+ Tempo[position].get(0) );
                        
                     }
                     counter*=2;
                     for(int c=0 ; c< counter ; c++)
                     {
                         Q.Data.add(Tempo[c]);
                         
                     }
                     Tree.add(Q);
                     if(Tree.get(Tree.size()-1).Data.size() < NumOfLevel)  // because i want to save last avg
                     {
                         avgSplit = new ArrayList<>();
                     }
                     
                     Q = new NonUniformQ();
                 }
             }
         }
//            for (int x=0; x<Tree.size() ; x++)
//            {
//                for(int c=0 ; c<Tree.get(x).Data.size() ; c++)
//                {   
//                    for(int k=0 ; k<Tree.get(x).Data.get(c).size() ; k++)
//                    {
//                           System.out.print(Tree.get(x).Data.get(c).get(k)+" - ");
//                           //System.out.print(avgSplitCheck.get(2) + " - ");
//                          
//                    }
//                           System.out.println();
//                           System.out.println("-------------------------------------");
//                }
//            } 
         
         Compression(table, val);
                    
      }
    public static void Compression(Table table , ArrayList<Double> val)
    {
        ArrayList<Double> CompressedData = new ArrayList<>();
        for(int i =0 ; i<val.size() ; i++)
        {
            for(int j=0 ; j<table.Q_1.size() ; j++)
            {
                if(( val.get(i)<table.higher.get(j))&&(val.get(i) >= table.lower.get(j)))
                {
                    CompressedData.add(table.Q.get(j));
                    break;
                }
            }
        }
//        System.out.println();
//        System.out.println("/*************************Compreesed Data **********************/");
//        for(int i=0 ; i<CompressedData.size() ; i++)
//        {
//            System.out.print( CompressedData.get(i) + "  ");
//        }
        /*************************************************write in file **************************************/
        try    //Put Compressed In File "Compressed.Txt"
               {
                   FileWriter FOut =new FileWriter("Compressed.txt");
                   for(int i=0; i<CompressedData.size() ;i++)
                   {
                       String COMPRESSION=""+(CompressedData.get(i));
                       FOut.write(COMPRESSION);
                       FOut.write("\n");
                   }
                   
                   FOut.close();
               }
               catch (Exception ex)
               {
                   System.out.println("Error Message");
               }
               
         try    //Put Table In File "Table.Txt"
               {
                   FileWriter FOut =new FileWriter("Table.txt");
                   BufferedWriter bufferedWriter =new BufferedWriter(FOut);
                   for(int i=0; i<table.Q.size() ;i++)
                   {
                       String s= ""+table.Q.get(i);
                       bufferedWriter.write(s);
                       bufferedWriter.write("_");
                       s= ""+table.Q_1.get(i);
                       bufferedWriter.write(s);
                       bufferedWriter.write("\n");
                   }
                   bufferedWriter.close();
               }
               catch (Exception ex)
               {
                   System.out.println("Error Message");
               }
    
     
    }
    public static ArrayList<Double> Decompression()
    {
        ArrayList<Double> Compressed = new ArrayList<>();
        ArrayList<Double> Decompressed = new ArrayList<>();
                try    //read Compressed from File "Compressed.Txt"
                    {
                        String pointer="";
                        File fileName =new File("Compressed.txt");
                        FileReader fileReader =  new FileReader(fileName);
                        BufferedReader bufferedReader =  new BufferedReader(fileReader);
                        while((pointer = bufferedReader.readLine()) != null)
                        {  
                            double y = Double.parseDouble(pointer); 
                            Compressed.add(y);
                           // System.out.println("POinter "+pointer );
                        }
                    }
                catch (Exception ex)
                      {
                        System.out.println("*-*Error Message");
                      }
        /****************************************read Table***************************************/
          Table Temo=new Table();
            try    
              {
                  String line=null;
                  String po="";          //to can split string and get pointer and next char
                  File fileName =new File("Table.txt");
                  FileReader fileReader =  new FileReader(fileName);
                  BufferedReader bufferedReader =  new BufferedReader(fileReader);
                  while((line = bufferedReader.readLine()) != null) 
                  {
                      for(int t=0; t<line.length();t++)
                      {
                          if(line.charAt(t)=='_')
                          {
                              double y = Double.parseDouble(po);
                              Temo.Q.add(y);
                              po="";
                          }
                          else
                          {
                               po+=line.charAt(t);
                          }
                      }    double y = Double.parseDouble(po);
                            Temo.Q_1.add(y);
                            po="";
                  }       
              }
              catch (Exception ex)
              {
                  System.out.println("*-*Error Message");
              }
//                  for(int i=0 ; i<Temo.Q.size() ; i++)
//                  {
//                      System.out.println("Temo.Q "+Temo.Q.get(i) +" Temo.Q_1 "+Temo.Q_1.get(i));
//       
//                  }    
        /****************************************Decompression********************************************/
        for(int i=0 ; i<Compressed.size() ; i++)
        {
            for(int j=0 ; j<Temo.Q.size() ; j++)
            {
                if (Objects.equals(Compressed.get(i), Temo.Q.get(j)))
                {
                    Decompressed.add(Temo.Q_1.get(j));
                }
            }
        }
        System.out.println();
        System.out.println("quantization.Quantization.Decompression()");
        for(int i=0 ; i<Decompressed.size() ; i++)
        {
            System.out.print(Decompressed.get(i)+ "  ");
        }
      //writeImage(Decompressed, "Decompressed Image.jpg" );
      return Decompressed;
    }
    public static void main(String[] args) throws IOException {
//        ArrayList<Double> img2matriz = readImage("D:\\java learning\\Quantization\\portrait-of-lynx-in-black-and-white-lukas-holas.jpg");
//        //int[][] img2matriz = readImage("D:\\java learning\\Quantization\\portrait-of-lynx-in-black-and-white-lukas-holas.jpg");
//        //writeImage(img2matriz, "Decompressed Image.jpg" );
//        ArrayList<Double>RealData=new ArrayList<>();
//        RealData.add(6.0);RealData.add(15.0);RealData.add(17.0);RealData.add(60.0);RealData.add(100.0);RealData.add(90.0);
//        RealData.add(66.0);RealData.add(59.0);RealData.add(18.0);RealData.add(3.0);RealData.add(5.0);RealData.add(16.0);
//        RealData.add(14.0);RealData.add(67.0);RealData.add(63.0);RealData.add(2.0);RealData.add(98.0);RealData.add(92.0);
//        //Min(RealData);
//        //GetAvg(RealData);
//        System.out.print("Enter number of level :");
//        int NumofLevel;
//        Scanner input = new Scanner(System.in);
//        NumofLevel= input.nextInt();
//        while(true)
//        {
//            if((NumofLevel%2==0 )&&((NumofLevel/2)%2==0))
//            {
//                break;
//            }
//            else
//            {
//                System.out.print("Invalid number of level :");
//                System.out.print("Enter number of level :");
//               NumofLevel= input.nextInt();
//            }
//        }
//        Quantization(img2matriz, NumofLevel);
//        Decompression();
//    
              NonUnformGui k=new NonUnformGui();
              k.setVisible(true) ;
}
}