# file-extension-identifier
A file with no extension is submitted for input. The program reads its signature and determines the file extension.

Source code:

import java.io.FileInputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.concurrent.TimeUnit;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;

public class Main {
    public static final HashMap<String, ArrayList<String>> list = new HashMap<>();

    public static void init() {
        list.put("png", new ArrayList<>(List.of("89 50 4E 47")));
        list.put("zip", new ArrayList<>(List.of("50 4B 03 04")));
        list.put("jpg", new ArrayList<>(List.of("FF D8 FF DB", "FF D8 FF E0", "FF D8 FF E1")));
        list.put("webp",new ArrayList<>(List.of("52 49 46 46")));
        list.put("doc", new ArrayList<>(List.of("0D 44 4F 43")));
        list.put("gif", new ArrayList<>(List.of("47 49 46 38")));
        list.put("pdf", new ArrayList<>(List.of("25 50 44 46")));
        list.put("RAR", new ArrayList<>(List.of("52 61 72 21")));
        list.put("mp3", new ArrayList<>(List.of("49 44 33")));
        list.put("mp4", new ArrayList<>(List.of("66 74 79 70")));
        list.put("7z", new ArrayList<>(List.of("37 7A BC AF")));
        list.put("iso", new ArrayList<>(List.of("43 44 30 30")));
        list.put("bmp", new ArrayList<>(List.of("42 4D")));

    }

    public static StringBuilder getExtension(Path file) {
        StringBuilder extension = new StringBuilder();
        int signatureLength = 4;
        try {

            byte[] signature = new byte[signatureLength];
            try (FileInputStream fis = new FileInputStream(file.toFile())) {
                fis.read(signature);
            }


            StringBuilder hexString = new StringBuilder();
            for (byte b : signature) {
                hexString.append(String.format("%02X ", b));
            }
            System.out.println(hexString);


            if (hexString.toString().trim().equals("50 4B 03 04")) {

                String specificType = checkZipContent(file);
                extension.append(specificType);
            } else {

                for (String key : list.keySet()) {
                    for (String val : list.get(key)) {
                        if (val.equals(hexString.toString().trim())){
                            extension.append(key);
                            break;
                        }
                    }
                }
            }
        } catch (IOException ignored) {
        }
        return extension;
    }

    private static String checkZipContent(Path file) {
        try (FileInputStream fis = new FileInputStream(file.toFile());
             ZipInputStream zis = new ZipInputStream(fis)) {

            ZipEntry entry;
            while ((entry = zis.getNextEntry()) != null) {
                String name = entry.getName();


                if (name.contains("word/document.xml")) {
                    return "docx";
                } else if (name.contains("xl/workbook.xml")) {
                    return "xlsx";
                } else if (name.contains("ppt/presentation.xml")) {
                    return "pptx";
                }
            }

            return "zip";
        } catch (IOException e) {

            return "zip";
        }
    }


    public static void changeExtension(Path file, StringBuilder extension) throws IOException, InterruptedException {
        String s = file.toString();
        String name = s.substring(s.lastIndexOf("\\")+1);
        Files.move(file, file.resolveSibling(name+"."+extension));
        System.out.println("Найдено расширение файла! "+extension+"\nМеняем расширение...");
        TimeUnit.SECONDS.sleep(2);
        System.out.println("Результат сохранён под именем: "+name+"."+getExtension(file)+extension);
    }


    public static void main(String[] args) throws IOException, InterruptedException {
        Path filePath = Paths.get(args[0]);
        init();
        StringBuilder ex = getExtension(filePath);
        changeExtension(filePath, ex);

    }
}
