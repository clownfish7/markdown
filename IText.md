### iText



pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.clownfish7</groupId>
    <artifactId>iText</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.itextpdf/itextpdf -->
        <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>itextpdf</artifactId>
            <version>5.5.13.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.itextpdf/itext-asian -->
        <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>itext-asian</artifactId>
            <version>5.2.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.7.0-M1</version>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.bouncycastle/bcprov-jdk15on -->
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcprov-jdk15on</artifactId>
            <version>1.65</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.bouncycastle/bcpkix-jdk15on -->
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcpkix-jdk15on</artifactId>
            <version>1.65</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```



code.java

```java
package com.clownfish7.itext;


import com.itextpdf.text.*;
import com.itextpdf.text.html.simpleparser.HTMLWorker;
import com.itextpdf.text.pdf.*;
import com.itextpdf.text.pdf.draw.DottedLineSeparator;
import com.itextpdf.text.pdf.draw.LineSeparator;
import com.itextpdf.text.pdf.draw.VerticalPositionMark;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInstance;

import java.io.File;
import java.io.FileOutputStream;
import java.io.StringReader;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.function.Consumer;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

/**
 * @author yzy
 * @classname PDFTest
 * @description TODO
 * @create 2020-05-14 16:33
 */
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class PDFTest {

    private static final String FILE_NAME = "clownfish7.pdf";
    private static final String FILE_PATH = "src\\test\\resources\\";
    private final File file = new File(FILE_PATH + FILE_NAME);
    Font font = FontFactory.getFont("STSong-Light", "UniGB-UCS2-H", BaseFont.NOT_EMBEDDED, 10f, Font.NORMAL, BaseColor.BLACK);

    //    @BeforeAll
    public void clean() {
        System.out.println("file clean starting!");
        file.deleteOnExit();
        System.out.println("file clean finished!");
    }


    @Test
    public void testCreatePDF() throws Exception {
        // 1. create document
        Document document = new Document();

        // 2. get a pdfWriter instance
        PdfWriter.getInstance(document, new FileOutputStream(file));

        // 3. open the document
        document.open();

        // 4. add content
        document.add(new Paragraph("hello world"));

        // 5. close the document
        document.close();
    }

    @Test
    public void testPDFProperties() throws Exception {
        // page size
        Rectangle rectangle = new Rectangle(PageSize.A4.rotate());
        // background color
        rectangle.setBackgroundColor(BaseColor.RED);
        //
        Document document = new Document(rectangle);
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));

        // PDF version (default 1.4)
        writer.setPdfVersion(PdfWriter.PDF_VERSION_1_2);

        // document properties
        document.addTitle("Title@clownfish7");
        document.addAuthor("Author@clownfish7");
        document.addSubject("Subject@clownfish7");
        document.addKeywords("Keywords@clownfish7");
        document.addCreator("Creator@clownfish7");

        // page margins
        document.setMargins(10, 20, 30, 40);

        document.open();
        document.add(new Paragraph("Hello World"));
        document.close();
    }

    @Test
    public void testPassword() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        // set password
        writer.setEncryption("Hello".getBytes(), "World".getBytes(),
                PdfWriter.ALLOW_SCREENREADERS,
                PdfWriter.STANDARD_ENCRYPTION_128);
        document.open();
        document.add(new Paragraph("Hello World"));
        document.close();
    }

    @Test
    public void testAddPage() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();
        document.add(new Paragraph("First page"));

        document.newPage();
        writer.setPageEmpty(false);

        document.newPage();
        document.add(new Paragraph("New page"));
        document.close();
    }

    @Test
    public void testAddWaterReflected() throws Exception {

        // image water reflected
        PdfReader reader = new PdfReader(FILE_PATH + FILE_NAME);
        PdfStamper stamp = new PdfStamper(reader, new FileOutputStream(FILE_PATH + "setWatermark2.pdf"));

        Image img = Image.getInstance(FILE_PATH + "head1.jpg");
        img.setAbsolutePosition(200, 400);
        PdfContentByte under = stamp.getUnderContent(1);
        under.addImage(img);

        // word water reflected
        PdfContentByte over = stamp.getOverContent(2);
        over.beginText();
        PdfGState gs = new PdfGState();
        gs.setFillOpacity(0.3f);// 设置透明度为0.8
        over.setGState(gs);
        BaseFont bf = BaseFont.createFont(BaseFont.HELVETICA, BaseFont.WINANSI, BaseFont.EMBEDDED);
        over.setFontAndSize(bf, 18);
        over.setTextMatrix(30, 30);
        over.showTextAligned(Element.ALIGN_LEFT, "DUPLICATE", 230, 430, 45);
        over.endText();


        // background image
        Image img2 = Image.getInstance(FILE_PATH + "head1.jpg");
        img2.setAbsolutePosition(0, 0);
        PdfContentByte under2 = stamp.getUnderContent(3);
        under2.addImage(img2);

        stamp.close();
        reader.close();
    }

    @Test
    public void testInsertChunk_Phrase_Paragraph_List() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();
        //Chunk对象: a String, a Font, and some attributes
        document.add(new Chunk("China"));
        document.add(new Chunk(" "));
        Font font = new Font(Font.FontFamily.HELVETICA, 6, Font.BOLD, BaseColor.WHITE);
        Chunk id = new Chunk("chinese", font);
        id.setBackground(BaseColor.BLACK, 1f, 0.5f, 1f, 1.5f);
        id.setTextRise(6);
        document.add(id);
        document.add(Chunk.NEWLINE);

        document.add(new Chunk("Japan"));
        document.add(new Chunk(" "));
        Font font2 = new Font(Font.FontFamily.HELVETICA, 6, Font.BOLD, BaseColor.WHITE);
        Chunk id2 = new Chunk("japanese", font2);
        id2.setBackground(BaseColor.BLACK, 1f, 0.5f, 1f, 1.5f);
        id2.setTextRise(6);
        id2.setUnderline(0.2f, -2f);
        document.add(id2);
        document.add(Chunk.NEWLINE);

        //Phrase对象: a List of Chunks with leading
        document.newPage();
        document.add(new Phrase("Phrase page"));

        Phrase director = new Phrase();
        Chunk name = new Chunk("China");
        name.setUnderline(0.2f, -2f);
        director.add(name);
        director.add(new Chunk(","));
        director.add(new Chunk(" "));
        director.add(new Chunk("chinese"));
        director.setLeading(24);
        document.add(director);

        Phrase director2 = new Phrase();
        Chunk name2 = new Chunk("Japan");
        name2.setUnderline(0.2f, -2f);
        director2.add(name2);
        director2.add(new Chunk(","));
        director2.add(new Chunk(" "));
        director2.add(new Chunk("japanese"));
        director2.setLeading(24);
        document.add(director2);

        //Paragraph对象: a Phrase with extra properties and a newline
        document.newPage();
        document.add(new Paragraph("Paragraph page"));

        Paragraph info = new Paragraph();
        info.add(new Chunk("China "));
        info.add(new Chunk("chinese"));
        info.add(Chunk.NEWLINE);
        info.add(new Phrase("Japan "));
        info.add(new Phrase("japanese"));
        document.add(info);

        //List对象: a sequence of Paragraphs called ListItem
        document.newPage();
        List list = new List(List.ORDERED);
        for (int i = 0; i < 10; i++) {
            ListItem item = new ListItem(String.format("%s: %d movies", "country" + (i + 1), (i + 1) * 100), new Font(Font.FontFamily.HELVETICA, 6, Font.BOLD, BaseColor.WHITE));
            List movielist = new List(List.ORDERED, List.ALPHABETICAL);
            movielist.setLowercase(List.LOWERCASE);
            for (int j = 0; j < 5; j++) {
                ListItem movieitem = new ListItem("Title" + (j + 1));
                List directorlist = new List(List.UNORDERED);
                for (int k = 0; k < 3; k++) {
                    directorlist.add(String.format("%s, %s", "Name1" + (k + 1),
                            "Name2" + (k + 1)));
                }
                movieitem.add(directorlist);
                movielist.add(movieitem);
            }
            item.add(movielist);
            list.add(item);
        }
        document.add(list);
        document.close();
    }

    @Test
    public void testInsertAnchor_Image_Chapter_Section() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        //Anchor对象: internal and external links
        Paragraph country = new Paragraph();
        Anchor dest = new Anchor("china", new Font(Font.FontFamily.HELVETICA, 14, Font.BOLD, BaseColor.BLUE));
        dest.setName("CN");
        dest.setReference("http://www.china.com");//external
        country.add(dest);
        country.add(String.format(": %d sites", 10000));
        document.add(country);

        document.newPage();
        Anchor toUS = new Anchor("Go to first page.", new Font(Font.FontFamily.HELVETICA, 14, Font.BOLD, BaseColor.BLUE));
        toUS.setReference("#CN");//internal
        document.add(toUS);

        //Image对象
        document.newPage();
        Image img = Image.getInstance(FILE_PATH + "head1.jpg");
        img.setAlignment(Image.LEFT | Image.TEXTWRAP);
        img.setBorder(Image.BOX);
        img.setBorderWidth(10);
        img.setBorderColor(BaseColor.WHITE);
        img.scaleToFit(1000, 72);//大小
        img.setRotationDegrees(-30);//旋转
        document.add(img);

        //Chapter, Section对象（目录）
        document.newPage();
        Paragraph title = new Paragraph("Title");
        Chapter chapter = new Chapter(title, 1);

        title = new Paragraph("Section A");
        Section section = chapter.addSection(title);
        section.setBookmarkTitle("bmk");
        section.setIndentation(30);
        section.setBookmarkOpen(false);
        section.setNumberStyle(
                Section.NUMBERSTYLE_DOTTED_WITHOUT_FINAL_DOT);

        Section subsection = section.addSection(new Paragraph("Sub Section A"));
        subsection.setIndentationLeft(20);
        subsection.setNumberDepth(1);

        document.add(chapter);
        document.close();
    }

    @Test // 画画
    public void testDraw() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        // 左右箭头
        document.add(new VerticalPositionMark() {

            public void draw(PdfContentByte canvas, float llx, float lly,
                             float urx, float ury, float y) {
                canvas.beginText();
                BaseFont bf = null;
                try {
                    bf = BaseFont.createFont(BaseFont.ZAPFDINGBATS, "", BaseFont.EMBEDDED);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                canvas.setFontAndSize(bf, 12);

                // LEFT
                canvas.showTextAligned(Element.ALIGN_CENTER, String.valueOf((char) 220), llx - 10, y, 0);
                // RIGHT
                canvas.showTextAligned(Element.ALIGN_CENTER, String.valueOf((char) 220), urx + 10, y + 8, 180);

                canvas.endText();
            }
        });

        // 直线
        Paragraph p1 = new Paragraph("LEFT");
        p1.add(new Chunk(new LineSeparator()));
        p1.add("R");
        document.add(p1);
        // 点线
        Paragraph p2 = new Paragraph("LEFT");
        p2.add(new Chunk(new DottedLineSeparator()));
        p2.add("R");
        document.add(p2);
        // 下滑线
        LineSeparator UNDERLINE = new LineSeparator(1, 100, null, Element.ALIGN_CENTER, -2);
        Paragraph p3 = new Paragraph("NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN");
        p3.add(UNDERLINE);
        document.add(p3);
        document.close();
    }

    public void test(Consumer<Document> consumer) {
        Document document = new Document();
        try {
            PdfWriter.getInstance(document, new FileOutputStream(file));
        } catch (Exception e) {
            e.printStackTrace();
        }
        document.open();
        consumer.accept(document);
        document.close();
    }

    @Test // 设置段落
    public void testSetParagraph() throws Exception {
        // 设置段落
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        Paragraph p = new Paragraph(
                "In the previous example, you added a header and footer with the showTextAligned() method. This example demonstrates that it’s sometimes more interesting to use PdfPTable and writeSelectedRows(). You can define a bottom border for each cell so that the header isunderlined. This is the most elegant way to add headers and footers, because the table mechanism allows you to position and align lines, images,and text.");

        //默认
        p.setAlignment(Element.ALIGN_JUSTIFIED);
        document.add(p);

        document.newPage();
        p.setAlignment(Element.ALIGN_JUSTIFIED);
        p.setIndentationLeft(1 * 15f);
        p.setIndentationRight((5 - 1) * 15f);
        document.add(p);

        //居右
        document.newPage();
        p.setAlignment(Element.ALIGN_RIGHT);
        p.setSpacingAfter(15f);
        document.add(p);

        //居左
        document.newPage();
        p.setAlignment(Element.ALIGN_LEFT);
        p.setSpacingBefore(15f);
        document.add(p);

        //居中
        document.newPage();
        p.setAlignment(Element.ALIGN_CENTER);
        p.setSpacingAfter(15f);
        p.setSpacingBefore(15f);
        document.add(p);
        document.close();
    }

    @Test // 删除页
    public void testDeletePage() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        document.open();
        document.add(new Paragraph("First page"));

        document.newPage();
        writer.setPageEmpty(false);

        document.newPage();
        document.add(new Paragraph("New page"));

        document.close();

        PdfReader reader = new PdfReader(FILE_PATH + FILE_NAME);
        reader.selectPages("1,3");
        PdfStamper stamp = new PdfStamper(reader, new FileOutputStream(FILE_PATH + "deletePage2.pdf"));
        stamp.close();
        reader.close();
    }

    @Test // 插入页
    public void testInsertPage() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        document.add(new Paragraph("1 page"));

        document.newPage();
        document.add(new Paragraph("2 page"));

        document.newPage();
        document.add(new Paragraph("3 page"));

        document.close();

        PdfReader reader = new PdfReader(FILE_PATH + FILE_NAME);
        PdfStamper stamp = new PdfStamper(reader, new FileOutputStream(FILE_PATH + "insertPage2.pdf"));

        stamp.insertPage(2, reader.getPageSize(1));

        ColumnText ct = new ColumnText(null);
        ct.addElement(new Paragraph(24, new Chunk("INSERT PAGE")));
        ct.setCanvas(stamp.getOverContent(2));
        ct.setSimpleColumn(36, 36, 559, 770);

        stamp.close();
        reader.close();

    }

    @Test // 排序
    public void testOrderPage() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));

        writer.setLinearPageMode();

        document.open();
        document.add(new Paragraph("1 page"));
        document.newPage();
        document.add(new Paragraph("2 page"));
        document.newPage();
        document.add(new Paragraph("3 page"));
        document.newPage();
        document.add(new Paragraph("4 page"));
        document.newPage();
        document.add(new Paragraph("5 page"));

        int[] order = {4, 3, 2, 1};
        writer.reorderPages(order);

        document.close();
    }

    @Test // 目录
    public void testDirectiry() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();
        // Code 1
        document.add(new Chunk("Chapter 1").setLocalDestination("1"));

        document.newPage();
        document.add(new Chunk("Chapter 2").setLocalDestination("2"));
        document.add(new Paragraph(new Chunk("Sub 2.1").setLocalDestination("2.1")));
        document.add(new Paragraph(new Chunk("Sub 2.2").setLocalDestination("2.2")));

        document.newPage();
        document.add(new Chunk("Chapter 3").setLocalDestination("3"));

        // Code 2
        PdfContentByte cb = writer.getDirectContent();
        PdfOutline root = cb.getRootOutline();

        // Code 3
        @SuppressWarnings("unused")
        PdfOutline oline1 = new PdfOutline(root, PdfAction.gotoLocalPage("1", false), "Chapter 1");

        PdfOutline oline2 = new PdfOutline(root, PdfAction.gotoLocalPage("2", false), "Chapter 2");
        oline2.setOpen(false);

        @SuppressWarnings("unused")
        PdfOutline oline2_1 = new PdfOutline(oline2, PdfAction.gotoLocalPage("2.1", false), "Sub 2.1");
        @SuppressWarnings("unused")
        PdfOutline oline2_2 = new PdfOutline(oline2, PdfAction.gotoLocalPage("2.2", false), "Sub 2.2");

        @SuppressWarnings("unused")
        PdfOutline oline3 = new PdfOutline(root, PdfAction.gotoLocalPage("3", false), "Chapter 3");

        document.close();
    }

    @Test // 页头页脚
    public void testHeader_Footer() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));

        writer.setPageEvent(new PdfPageEventHelper() {

            public void onEndPage(PdfWriter writer, Document document) {

                PdfContentByte cb = writer.getDirectContent();
                cb.saveState();

                cb.beginText();
                BaseFont bf = null;
                try {
                    bf = BaseFont.createFont(BaseFont.HELVETICA, BaseFont.WINANSI, BaseFont.EMBEDDED);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                cb.setFontAndSize(bf, 10);

                //Header
                float x = document.top(-20);

                //左
                cb.showTextAligned(PdfContentByte.ALIGN_LEFT,
                        "H-Left",
                        document.left(), x, 0);
                //中
                cb.showTextAligned(PdfContentByte.ALIGN_CENTER,
                        writer.getPageNumber() + " page",
                        (document.right() + document.left()) / 2,
                        x, 0);
                //右
                cb.showTextAligned(PdfContentByte.ALIGN_RIGHT,
                        "H-Right",
                        document.right(), x, 0);

                //Footer
                float y = document.bottom(-20);

                //左
                cb.showTextAligned(PdfContentByte.ALIGN_LEFT,
                        "F-Left",
                        document.left(), y, 0);
                //中
                cb.showTextAligned(PdfContentByte.ALIGN_CENTER,
                        writer.getPageNumber() + " page",
                        (document.right() + document.left()) / 2,
                        y, 0);
                //右
                cb.showTextAligned(PdfContentByte.ALIGN_RIGHT,
                        "F-Right",
                        document.right(), y, 0);

                cb.endText();

                cb.restoreState();
            }
        });

        document.open();
        document.add(new Paragraph("1 page"));
        document.newPage();
        document.add(new Paragraph("2 page"));
        document.newPage();
        document.add(new Paragraph("3 page"));
        document.newPage();
        document.add(new Paragraph("4 page"));
        document.close();
    }

    @Test // 左右文字
    public void testText_Left_Right() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        PdfContentByte canvas = writer.getDirectContent();

        Phrase phrase1 = new Phrase("This is a test!left");
        Phrase phrase2 = new Phrase("This is a test!right");
        Phrase phrase3 = new Phrase("This is a test!center");
        ColumnText.showTextAligned(canvas, Element.ALIGN_LEFT, phrase1, 100, 500, 0);
        ColumnText.showTextAligned(canvas, Element.ALIGN_RIGHT, phrase2, 100, 536, 0);
        ColumnText.showTextAligned(canvas, Element.ALIGN_CENTER, phrase3, 100, 572, 0);

        document.close();
    }

    @Test // 幻灯片播放
    public void testPPT() throws Exception {
        // 幻灯片放映
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));

        writer.setPdfVersion(PdfWriter.VERSION_1_5);

        writer.setViewerPreferences(PdfWriter.PageModeFullScreen);//全屏
        writer.setPageEvent(new PdfPageEventHelper() {
            public void onStartPage(PdfWriter writer, Document document) {
                writer.setTransition(new PdfTransition(PdfTransition.DISSOLVE, 3));
                writer.setDuration(5);//间隔时间
            }
        });

        document.open();
        document.add(new Paragraph("1 page"));
        document.newPage();
        document.add(new Paragraph("2 page"));
        document.newPage();
        document.add(new Paragraph("3 page"));
        document.newPage();
        document.add(new Paragraph("4 page"));
        document.newPage();
        document.add(new Paragraph("5 page"));

        document.close();
    }

    @Test // zip
    public void testZip() throws Exception {
        ZipOutputStream zip = new ZipOutputStream(new FileOutputStream(FILE_PATH + "zipPDF.zip"));
        for (int i = 1; i <= 3; i++) {
            ZipEntry entry = new ZipEntry("hello_" + i + ".pdf");
            zip.putNextEntry(entry);

            Document document = new Document();
            PdfWriter writer = PdfWriter.getInstance(document, zip);
            writer.setCloseStream(false);
            document.open();
            document.add(new Paragraph("Hello " + i));
            document.close();
            zip.closeEntry();
        }
        zip.close();
    }

    @Test // 分割PDF
    public void testPdfSplit() throws Exception {
        FileOutputStream out = new FileOutputStream(FILE_PATH + "splitPDF.pdf");

        Document document = new Document();

        PdfWriter.getInstance(document, out);

        document.open();
        document.add(new Paragraph("1 page"));

        document.newPage();
        document.add(new Paragraph("2 page"));

        document.newPage();
        document.add(new Paragraph("3 page"));

        document.newPage();
        document.add(new Paragraph("4 page"));

        document.close();

        PdfReader reader = new PdfReader(FILE_PATH + "splitPDF.pdf");

        Document dd = new Document();
        PdfWriter writer = PdfWriter.getInstance(dd, new FileOutputStream(FILE_PATH + "splitPDF1.pdf"));
        dd.open();
        PdfContentByte cb = writer.getDirectContent();
        dd.newPage();
        cb.addTemplate(writer.getImportedPage(reader, 1), 0, 0);
        dd.newPage();
        cb.addTemplate(writer.getImportedPage(reader, 2), 0, 0);
        dd.close();
        writer.close();

        Document dd2 = new Document();
        PdfWriter writer2 = PdfWriter.getInstance(dd2, new FileOutputStream(FILE_PATH + "splitPDF2.pdf"));
        dd2.open();
        PdfContentByte cb2 = writer2.getDirectContent();
        dd2.newPage();
        cb2.addTemplate(writer2.getImportedPage(reader, 3), 0, 0);
        dd2.newPage();
        cb2.addTemplate(writer2.getImportedPage(reader, 4), 0, 0);
        dd2.close();
        writer2.close();
    }

    @Test // 合并PDF
    public void testPdfMerge() throws Exception {
        PdfReader reader1 = new PdfReader(FILE_PATH + "splitPDF1.pdf");
        PdfReader reader2 = new PdfReader(FILE_PATH + "splitPDF2.pdf");

        FileOutputStream out = new FileOutputStream(FILE_PATH + "mergePDF.pdf");

        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, out);

        document.open();
        PdfContentByte cb = writer.getDirectContent();

        int totalPages = 0;
        totalPages += reader1.getNumberOfPages();
        totalPages += reader2.getNumberOfPages();

        java.util.List<PdfReader> readers = new ArrayList<>();
        readers.add(reader1);
        readers.add(reader2);

        int pageOfCurrentReaderPDF = 0;
        Iterator<PdfReader> iteratorPDFReader = readers.iterator();

        // Loop through the PDF files and add to the output.  
        while (iteratorPDFReader.hasNext()) {
            PdfReader pdfReader = iteratorPDFReader.next();

            // Create a new page in the target for each source page.  
            while (pageOfCurrentReaderPDF < pdfReader.getNumberOfPages()) {
                document.newPage();
                pageOfCurrentReaderPDF++;
                PdfImportedPage page = writer.getImportedPage(pdfReader, pageOfCurrentReaderPDF);
                cb.addTemplate(page, 0, 0);
            }
            pageOfCurrentReaderPDF = 0;
        }
        out.flush();
        document.close();
        out.close();
    }

    @Test // 注解
    public void testAnnotation() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        writer.setLinearPageMode();

        document.open();
        document.add(new Paragraph("1 page"));
        document.add(new Annotation("Title", "This is a annotation!"));

        document.newPage();
        document.add(new Paragraph("2 page"));
        Chunk chunk = new Chunk("\u00a0");
        chunk.setAnnotation(PdfAnnotation.createText(writer, null, "Title", "This is a another annotation!", false, "Comment"));
        document.add(chunk);

        //添加附件
        document.newPage();
        document.add(new Paragraph("3 page"));
        Chunk chunk2 = new Chunk("\u00a0\u00a0");
        PdfAnnotation annotation = PdfAnnotation.createFileAttachment(
                writer, null, "Title", null,
                FILE_PATH + "head1.jpg",
                FILE_PATH + "head2.jpg");
        annotation.put(PdfName.NAME, new PdfString("Paperclip"));
        chunk2.setAnnotation(annotation);
        document.add(chunk2);
        document.close();
    }

    @Test // 插入table
    public void testTable() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        PdfPTable table = new PdfPTable(3);
        PdfPCell cell;
        cell = new PdfPCell(new Phrase("Cell with colspan 3"));
        cell.setColspan(3);
        table.addCell(cell);
        cell = new PdfPCell(new Phrase("Cell with rowspan 2"));
        cell.setRowspan(2);
        table.addCell(cell);
        table.addCell("row 1; cell 1");
        table.addCell("row 1; cell 2");
        table.addCell("row 2; cell 1");
        table.addCell("row 2; cell 2");

        document.add(table);
        document.close();
    }

    @Test // 表格嵌套
    public void testTableNest() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        PdfPTable table = new PdfPTable(4);

        // 1行2列
        PdfPTable nested1 = new PdfPTable(2);
        nested1.addCell("1.1");
        nested1.addCell("1.2");

        // 2行1列
        PdfPTable nested2 = new PdfPTable(1);
        nested2.addCell("2.1");
        nested2.addCell("2.2");

        // 将表格插入到指定位置
        for (int k = 0; k < 24; ++k) {
            if (k == 1) {
                table.addCell(nested1);
            } else if (k == 20) {
                table.addCell(nested2);
            } else {
                table.addCell("cell " + k);
            }
        }

        document.add(table);
        document.close();
    }

    @Test // 设置表格宽度
    public void testSetTableWeight() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        PdfPTable table = new PdfPTable(3);
        PdfPCell cell;
        cell = new PdfPCell(new Phrase("Cell with colspan 3"));
        cell.setColspan(3);
        table.addCell(cell);
        cell = new PdfPCell(new Phrase("Cell with rowspan 2"));
        cell.setRowspan(2);
        table.addCell(cell);
        table.addCell("row 1; cell 1");
        table.addCell("row 1; cell 2");
        table.addCell("row 2; cell 1");
        table.addCell("row 2; cell 2");

        // 100%
        table.setWidthPercentage(100);
        document.add(table);
        document.add(new Paragraph("\n\n"));

        // 宽度50% 居左
        table.setWidthPercentage(50);
        table.setHorizontalAlignment(Element.ALIGN_LEFT);
        document.add(table);
        document.add(new Paragraph("\n\n"));

        // 宽度50% 居中
        table.setWidthPercentage(50);
        table.setHorizontalAlignment(Element.ALIGN_CENTER);
        document.add(table);
        document.add(new Paragraph("\n\n"));

        // 宽度50% 居右
        table.setWidthPercentage(50);
        table.setHorizontalAlignment(Element.ALIGN_RIGHT);
        document.add(table);
        document.add(new Paragraph("\n\n"));

        // 固定宽度
        table.setTotalWidth(300);
        table.setLockedWidth(true);
        document.add(table);

        document.close();
    }

    @Test // 设置表格前后间隔
    public void testSetTable_Before_After_Interval() throws Exception {

        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        PdfPTable table = new PdfPTable(3);
        PdfPCell cell = new PdfPCell(new Phrase("合并3个单元格"));
        cell.setColspan(3);
        table.addCell(cell);
        table.addCell("1.1");
        table.addCell("2.1");
        table.addCell("3.1");
        table.addCell("1.2");
        table.addCell("2.2");
        table.addCell("3.2");

        cell = new PdfPCell(new Phrase("红色边框", font));
        cell.setBorderColor(new BaseColor(255, 0, 0));
        table.addCell(cell);

        cell = new PdfPCell(new Phrase("合并单2个元格", font));
        cell.setColspan(2);
        cell.setBackgroundColor(new BaseColor(0xC0, 0xC0, 0xC0));
        table.addCell(cell);

        table.setWidthPercentage(50);

        document.add(new Paragraph("追加2个表格", font));
        document.add(table);
        document.add(table);

        document.newPage();
        document.add(new Paragraph("使用'SpacingBefore'和'setSpacingAfter'", font));
        table.setSpacingBefore(15f);
        document.add(table);
        document.add(table);
        document.add(new Paragraph("这里没有间隔", font));
        table.setSpacingAfter(15f);

        document.close();
    }

    @Test // 设置单元格宽度
    public void testSetCellWeight() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        // 按比例设置单元格宽度
        float[] widths = {0.1f, 0.1f, 0.05f, 0.75f};
        PdfPTable table = new PdfPTable(widths);
        table.addCell("10%");
        table.addCell("10%");
        table.addCell("5%");
        table.addCell("75%");
        table.addCell("aa");
        table.addCell("aa");
        table.addCell("a");
        table.addCell("aaaaaaaaaaaaaaa");
        table.addCell("bb");
        table.addCell("bb");
        table.addCell("b");
        table.addCell("bbbbbbbbbbbbbbb");
        table.addCell("cc");
        table.addCell("cc");
        table.addCell("c");
        table.addCell("ccccccccccccccc");
        document.add(table);
        document.add(new Paragraph("\n\n"));

        // 调整比例
        widths[0] = 20f;
        widths[1] = 20f;
        widths[2] = 10f;
        widths[3] = 50f;
        table.setWidths(widths);
        document.add(table);

        // 按绝对值设置单元格宽度
        widths[0] = 40f;
        widths[1] = 40f;
        widths[2] = 20f;
        widths[3] = 300f;
        Rectangle r = new Rectangle(PageSize.A4.getRight(72), PageSize.A4.getTop(72));
        table.setWidthPercentage(widths, r);
        document.add(new Paragraph("\n\n"));
        document.add(table);

        document.close();
    }

    @Test // 设置单元格高度
    public void testSetCellHeight() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        PdfPTable table = new PdfPTable(2);

        PdfPCell cell;

        // 折行
        table.addCell(new PdfPCell(new Paragraph("折行", font)));
        cell = new PdfPCell(new Paragraph("blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah"));
        cell.setNoWrap(false);
        table.addCell(cell);

        // 不折行
        table.addCell(new PdfPCell(new Paragraph("不折行", font)));
        cell.setNoWrap(true);
        table.addCell(cell);

        // 设置高度
        table.addCell(new PdfPCell(new Paragraph("任意高度", font)));
        cell = new PdfPCell(new Paragraph("1. blah blah\n2. blah blah blah\n3. blah blah\n4. blah blah blah\n5. blah blah\n6. blah blah blah\n7. blah blah\n8. blah blah blah"));
        table.addCell(cell);

        // 固定高度
        table.addCell(new PdfPCell(new Paragraph("固定高度", font)));
        cell.setFixedHeight(50f);
        table.addCell(cell);

        // 最小高度
        table.addCell(new PdfPCell(new Paragraph("最小高度", font)));
        cell = new PdfPCell(new Paragraph("最小高度：50", font));
        cell.setMinimumHeight(50f);
        table.addCell(cell);

        // 最后一行拉长到page底部
        table.setExtendLastRow(true);
        table.addCell(new PdfPCell(new Paragraph("拉长最后一行", font)));
        cell = new PdfPCell(new Paragraph("最后一行拉长到page底部", font));
        table.addCell(cell);

        document.add(table);

        document.close();
    }

    @Test // 设置单元格颜色
    public void testSetCellColor() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        PdfPTable table = new PdfPTable(4);
        PdfPCell cell;
        cell = new PdfPCell(new Paragraph("颜色测试", font));
        table.addCell(cell);

        // 红色背景，无边框
        cell = new PdfPCell(new Paragraph("红色背景，无边框", font));
        cell.setBorder(Rectangle.NO_BORDER);
        cell.setBackgroundColor(BaseColor.RED);
        table.addCell(cell);

        // 绿色背景，下边框
        cell = new PdfPCell(new Paragraph("绿色背景，下边框", font));
        cell.setBorder(Rectangle.BOTTOM);
        cell.setBorderColorBottom(BaseColor.MAGENTA);
        cell.setBorderWidthBottom(5f);
        cell.setBackgroundColor(BaseColor.GREEN);
        table.addCell(cell);

        // 蓝色背景，上边框
        cell = new PdfPCell(new Paragraph("蓝色背景，上边框", font));
        cell.setBorder(Rectangle.TOP);
        cell.setUseBorderPadding(true);
        cell.setBorderWidthTop(5f);
        cell.setBorderColorTop(BaseColor.CYAN);
        cell.setBackgroundColor(BaseColor.BLUE);
        table.addCell(cell);

        cell = new PdfPCell(new Paragraph("背景灰色度", font));
        table.addCell(cell);
        cell = new PdfPCell(new Paragraph("0.25"));
        cell.setBorder(Rectangle.NO_BORDER);
        cell.setGrayFill(0.25f);
        table.addCell(cell);
        cell = new PdfPCell(new Paragraph("0.5"));
        cell.setBorder(Rectangle.NO_BORDER);
        cell.setGrayFill(0.5f);
        table.addCell(cell);
        cell = new PdfPCell(new Paragraph("0.75"));
        cell.setBorder(Rectangle.NO_BORDER);
        cell.setGrayFill(0.75f);
        table.addCell(cell);

        document.add(table);

        document.close();
    }

    @Test // 插入图像
    public void testInsertImage() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();
        Image image = Image.getInstance(FILE_PATH + "head1.jpg");
        float[] widths = {1f, 4f};

        PdfPTable table = new PdfPTable(widths);

        // 插入图片
        table.addCell(new PdfPCell(new Paragraph("图片测试", font)));
        table.addCell(image);

        // 调整图片大小
        table.addCell("This two");
        table.addCell(new PdfPCell(image, true));

        // 不调整
        table.addCell("This three");
        table.addCell(new PdfPCell(image, false));
        document.add(table);

        document.close();
    }

    @Test // 设置表头
    public void testTableTitle() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        String[] bogusData = {"M0065920", "SL", "FR86000P", "PCGOLD",
                "119000", "96 06", "2001-08-13", "4350", "6011648299",
                "FLFLMTGP", "153", "119000.00"};
        int NumColumns = 12;
        // 12
        PdfPTable datatable = new PdfPTable(NumColumns);
        int headerwidths[] = {9, 4, 8, 10, 8, 11, 9, 7, 9, 10, 4, 10}; // percentage
        datatable.setWidths(headerwidths);
        datatable.setWidthPercentage(100);
        datatable.getDefaultCell().setPadding(3);
        datatable.getDefaultCell().setBorderWidth(2);
        datatable.getDefaultCell().setHorizontalAlignment(Element.ALIGN_CENTER);

        datatable.addCell("Clock #");
        datatable.addCell("Trans Type");
        datatable.addCell("Cusip");
        datatable.addCell("Long Name");
        datatable.addCell("Quantity");
        datatable.addCell("Fraction Price");
        datatable.addCell("Settle Date");
        datatable.addCell("Portfolio");
        datatable.addCell("ADP Number");
        datatable.addCell("Account ID");
        datatable.addCell("Reg Rep ID");
        datatable.addCell("Amt To Go ");

        datatable.setHeaderRows(1);

        // 边框
        datatable.getDefaultCell().setBorderWidth(1);

        // 背景色
        for (int i = 1; i < 1000; i++) {
            for (int x = 0; x < NumColumns; x++) {
                datatable.addCell(bogusData[x]);
            }
        }

        document.add(datatable);

        document.close();
    }

    @Test // 分割表格
    public void testTableSplit() throws Exception {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        //横向分割
        PdfContentByte cb = writer.getDirectContent();
        PdfPTable table = new PdfPTable(10);
        for (int k = 1; k <= 100; ++k) {
            table.addCell("The number " + k);
        }
        table.setTotalWidth(400);

        table.writeSelectedRows(0, 5, 0, -1, 5, 700, cb);
        table.writeSelectedRows(5, -1, 0, -1, 210, 700, cb);

        document.close();
    }

    @Test // 设置单元格留白
    public void testCellBlank() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        PdfPTable table = new PdfPTable(2);
        PdfPCell cell;
        Paragraph p = new Paragraph("Quick brown fox jumps over the lazy dog. Quick brown fox jumps over the lazy dog.");
        table.addCell(new PdfPCell(new Paragraph("默认", font)));
        table.addCell(p);
        table.addCell(new PdfPCell(new Paragraph("Padding：10", font)));
        cell = new PdfPCell(p);
        cell.setPadding(10f);
        cell.setBorder(0);
        table.addCell(cell);
        table.addCell(new PdfPCell(new Paragraph("Padding：0", font)));
        cell = new PdfPCell(p);
        cell.setPadding(0f);
        cell.setBorder(0);
        table.addCell(cell);
        table.addCell(new PdfPCell(new Paragraph("上Padding：0 左Padding：20", font)));
        cell = new PdfPCell(p);
        cell.setBorder(0);
        cell.setPaddingTop(0f);
        cell.setPaddingLeft(20f);
        table.addCell(cell);
        document.add(table);

        document.newPage();
        table = new PdfPTable(2);
        table.addCell(new PdfPCell(new Paragraph("没有Leading", font)));
        table.getDefaultCell().setLeading(0f, 0f);
        table.addCell("blah blah\nblah blah blah\nblah blah\nblah blah blah\nblah blah\nblah blah blah\nblah blah\nblah blah blah\n");
        table.getDefaultCell().setLeading(14f, 0f);
        table.addCell(new PdfPCell(new Paragraph("固定Leading：14pt", font)));
        table.addCell("blah blah\nblah blah blah\nblah blah\nblah blah blah\nblah blah\nblah blah blah\nblah blah\nblah blah blah\n");
        table.addCell(new PdfPCell(new Paragraph("相对于字体", font)));
        table.getDefaultCell().setLeading(0f, 1.0f);
        table.addCell("blah blah\nblah blah blah\nblah blah\nblah blah blah\nblah blah\nblah blah blah\nblah blah\nblah blah blah\n");
        document.add(table);

        document.close();
    }

    @Test // 设置单元格边框
    public void testCellBorder() throws Exception {
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();

        // 没有边框
        PdfPTable table1 = new PdfPTable(3);
        table1.getDefaultCell().setBorder(PdfPCell.NO_BORDER);
        table1.addCell(new Paragraph("Cell 1"));
        table1.addCell(new Paragraph("Cell 2"));
        table1.addCell(new Paragraph("Cell 3"));
        document.add(table1);

        // 边框粗细颜色
        document.newPage();
        Rectangle b1 = new Rectangle(0f, 0f);
        b1.setBorderWidthLeft(6f);
        b1.setBorderWidthBottom(5f);
        b1.setBorderWidthRight(4f);
        b1.setBorderWidthTop(2f);
        b1.setBorderColorLeft(BaseColor.RED);
        b1.setBorderColorBottom(BaseColor.ORANGE);
        b1.setBorderColorRight(BaseColor.YELLOW);
        b1.setBorderColorTop(BaseColor.GREEN);
        PdfPTable table2 = new PdfPTable(1);
        PdfPCell cell = new PdfPCell(new Paragraph("Cell 1"));
        cell.cloneNonPositionParameters(b1);
        table2.addCell(cell);
        document.add(table2);

        document.close();
    }

    @Test // 条形码 & 二维码
    public void testBarcode_QRcode() throws Exception {
        String myString = "http://www.google.com";
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(file));

        document.open();
        PdfContentByte pdfContentByte = writer.getDirectContent();

        Barcode128 code128 = new Barcode128();
        code128.setCode(myString.trim());
        code128.setCodeType(Barcode128.CODE128);
        Image code128Image = code128.createImageWithBarcode(pdfContentByte, null, null);
        code128Image.setAbsolutePosition(10, 700);
        code128Image.scalePercent(125);
        document.add(code128Image);

        BarcodeQRCode qrcode = new BarcodeQRCode(myString.trim(), 1, 1, null);
        Image qrcodeImage = qrcode.getImage();
        qrcodeImage.setAbsolutePosition(10, 600);
        qrcodeImage.scalePercent(200);
        document.add(qrcodeImage);
        document.close();
    }

    @Test // html -> pdf
    public void testHtml2Pdf() throws Exception {
        Document document = new Document(PageSize.LETTER);
        PdfWriter.getInstance(document, new FileOutputStream(file));
        document.open();
        HTMLWorker htmlWorker = new HTMLWorker(document);
        htmlWorker.parse(new StringReader("<h1>This is a test!</h1>"));
        document.close();
    }
}

```

