# Project-Error-Correcting-Encoder-Decoder
This is my implementetion of Jet Brains Error-Correcting-Encoder-Decoder project stage 5/5.

import javax.xml.transform.sax.SAXSource;

import java.io.*;

import java.util.Arrays;

import java.util.Random;

import java.util.Scanner;

import java.util.StringTokenizer;

public class Main {

    public static void main(String[] args) throws IOException {
        Scanner scan = new Scanner(System.in);
        switch (scan.nextLine()) {

            case "encode":
                encodeIt("send.txt", "encoded.txt");
                break;

            case "send":
                sendIt("encoded.txt", "received.txt");
                break;

            case "decode":
                decodeIt("received.txt", "decoded.txt");
                break;
            default:
                break;
        }
        scan.close();
    }

    static void encodeIt(String inFile, String outFile) throws IOException {
        try (FileInputStream inputStream = new FileInputStream(inFile);
             FileOutputStream fileOutputStream = new FileOutputStream(outFile, false);) {
            byte[] originalBytes = inputStream.readAllBytes();
            StringBuilder sb = new StringBuilder();

            for (byte b : originalBytes) {
                sb.append(String.format("%08d", Integer.parseInt(Integer.toBinaryString(b))));
            }
            for (int i = 0; i < sb.length(); i += 4) {
                StringBuilder sb1 = new StringBuilder();
                String auxA = sb.substring(i, i + 4);
                System.out.print(auxA + " ");

                String[] auxB = {"x", "x", String.valueOf(auxA.charAt(0)), "x", String.valueOf(auxA.charAt(1)),
                        String.valueOf(auxA.charAt(2)), String.valueOf(auxA.charAt(3)), "0"};

                auxB[0] = String.valueOf((Integer.parseInt(auxB[2]) + Integer.parseInt(auxB[4]) + Integer.parseInt(auxB[6])) % 2);
                auxB[1] = String.valueOf((Integer.parseInt(auxB[2]) + Integer.parseInt(auxB[5]) + Integer.parseInt(auxB[6])) % 2);
                auxB[3] = String.valueOf((Integer.parseInt(auxB[4]) + Integer.parseInt(auxB[5]) + Integer.parseInt(auxB[6])) % 2);

                for (String s : auxB) {
                    sb1.append(s);
                }

                String aux4 = String.valueOf(sb1);
                System.out.print((byte) Integer.parseInt(aux4, 2) + " ");
                fileOutputStream.write((byte) Integer.parseInt(aux4, 2));
                System.out.println();

            }
        }
    }

    static void sendIt(String inFile, String outFile) throws IOException {
        try (FileInputStream inputStream = new FileInputStream(inFile);
             FileOutputStream fileOutputStream = new FileOutputStream(outFile, false);) {

            byte[] byteS = inputStream.readAllBytes();
            for (byte b : byteS) {
                Random random = new Random();
                b ^= 1 << random.nextInt(8);
                fileOutputStream.write(b);
            }

        }

    }

    static void decodeIt(String inFile, String outFile) throws IOException {
        try (FileInputStream inputStream = new FileInputStream(inFile);
             FileOutputStream fileOutputStream = new FileOutputStream(outFile, false);) {
            byte[] byteS = inputStream.readAllBytes();
            StringBuilder sb = new StringBuilder();
            StringBuilder sb3 = new StringBuilder();
            for (byte b : byteS) {
                String aux = "";
                if (b >= 0) {
                    aux = String.format("%08d", Integer.parseInt(Integer.toBinaryString(b)));
                } else {
                    aux = Integer.toBinaryString(b).substring(24);
                }
                sb.append(aux);
            }

            for (int i = 0; i < sb.length(); i += 8) {
                StringBuilder sb1 = new StringBuilder();
                String[] aux = sb.substring(i, i + 7).split("");
                int pos = 0;
                if (Integer.parseInt(aux[0]) != (Integer.parseInt(aux[2]) +
                        Integer.parseInt(aux[4]) + Integer.parseInt(aux[6])) % 2) {
                    pos += 1;
                }

                if (Integer.parseInt(aux[1]) != (Integer.parseInt(aux[2]) +
                        Integer.parseInt(aux[5]) + Integer.parseInt(aux[6])) % 2) {
                    pos += 2;
                }

                if (Integer.parseInt(aux[3]) != (Integer.parseInt(aux[4]) +
                        Integer.parseInt(aux[5]) + Integer.parseInt(aux[6])) % 2) {
                    pos += 4;
                }

                if (pos > 1) {
                    if (Integer.parseInt(aux[pos - 1]) == 0) {
                        aux[pos - 1] = "1";
                    } else {
                        aux[pos - 1] = "0";
                    }
                }
                for (String s : aux) {
                    sb1.append(s);
                }

                sb3.append(sb1.charAt(2));
                sb3.append(sb1.charAt(4));
                sb3.append(sb1.charAt(5));
                sb3.append(sb1.charAt(6));
            }
            for (int i = 0; i < sb3.length(); i += 8) {
                fileOutputStream.write((char) Integer.parseInt(sb3.substring(i, i + 8), 2));
            }

        }

    }
}
