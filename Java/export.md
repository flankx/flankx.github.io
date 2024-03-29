# 导出文件

## 一、java导出数据到`EXCEL`

+ 1 引入依赖

````xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>${version}</version>
</dependency>
````

+ 2 写`EXCEL`

````java
try{
    response.setContentType("application/vnd.ms-excel");
    response.setCharacterEncoding(StandardCharsets.UTF_8.name());
    fileName = URLEncoder.encode(fileName, StandardCharsets.UTF_8.name());
    response.setHeader("Content-disposition", "attachment;filename=" + fileName + ".xlsx");
    EasyExcel.write(response.getOutputStream()).head(head).sheet(sheetName).doWrite(dataList);
} catch (Exception e) {
    log.error("export error! " + e);
}
````

## 二、java导出数据到`WORD`

+ 1 引入依赖

````xml
<!-- https://mvnrepository.com/artifact/org.apache.poi/poi -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>${version}</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>${version}</version>
</dependency>
<!-- poi-tl基于poi的word模板引擎 -->
<dependency>
    <groupId>com.deepoove</groupId>
    <artifactId>poi-tl</artifactId>
    <version>${version}</version>
</dependency>
</dependency>
````

+ 2 写`WORD`

````java
try{
    response.setContentType("application/msword");
    response.setCharacterEncoding(StandardCharsets.UTF_8.name());
    fileName = URLEncoder.encode(fileName, StandardCharsets.UTF_8.name());
    response.setHeader("Content-disposition", "attachment;filename=" + fileName + ".docx");

    XWPFDocument doc = new XWPFDocument();
    // 页眉处理
    // handleDocHeader(doc);
    // 页尾处理
    // handleDocFoot(doc);
    // 创建一个表格，并指定宽度
    XWPFTable table = doc.createTable(headSize + dataSize, head.size());
    // 表格宽度
    TableTools.widthTable(table, MiniTableRenderData.WIDTH_A4_FULL, 4);
    // 设置表格居中
    TableStyle style = new TableStyle();
    style.setAlign(STJc.CENTER);
    TableTools.styleTable(table, style);

    List<XWPFTableCell> row0 = table.getRow(0).getTableCells();
    row0.get(0).setText("第一列");
    row0.get(1).setText("第二列");
    row0.get(2).setText("第三列");
    // 写入流
    try (ServletOutputStream outputStream = response.getOutputStream()) {
        doc.write(outputStream);
    }
} catch (Exception e) {
    log.error("export error! " + e);
}
````

## 三、java导出数据到`PDF`

+ 1 引入依赖

````xml
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
    <version>${version}</version>
</dependency>
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itext-asian</artifactId>
    <version>${version}</version>
</dependency>
````

+ 2 写`PDF`

````java
try {
    response.setContentType("application/pdf");
    response.setCharacterEncoding(StandardCharsets.UTF_8.name());
    fileName = URLEncoder.encode(fileName, StandardCharsets.UTF_8.name());
    response.setHeader("Content-disposition", "attachment;filename=" + fileName + ".pdf");

    // 新建document对象
    Document document = new Document(PageSize.A4);
    // 创建 PdfWriter 对象
    PdfWriter.getInstance(document, response.getOutputStream());
    // 获取图片字节数组
    Image image = getLogoImage(document, IOUtils.toByteArray(inputStream));
    // 页眉页脚处理器
    PDFHeadFootEventHandler eventHandler = new PDFHeadFootEventHandler(image, 6);
    pdfWriter.setPageEvent(eventHandler);
    // 打开文档
    document.open();
    // pdf文档中中文字体的设置，注意一定要添加iTextAsian.jar包
    BaseFont bfChinese = BaseFont.createFont("STSong-Light", "UniGB-UCS2-H", BaseFont.NOT_EMBEDDED);
    // 加入document：
    Font fontChinese = new Font(bfChinese, 12, Font.BOLD);
    // 创建表格对象（包含行列矩阵的表格）;
    PdfPTable table = new PdfPTable(size);
    table.setPaddingTop(5);
    table.setSpacingAfter(5);
    // 表格总宽度
    table.setTotalWidth(400);
    // 锁定宽度
    table.setLockedWidth(true);
    // 列
    PdfPCell cell1 = new PdfPCell(new Paragraph("第一列", fontChinese));
    table.addCell(cell1);
    PdfPCell cell2 = new PdfPCell(new Paragraph("第二列", fontChinese));
    table.addCell(cell2);
    PdfPCell cell3 = new PdfPCell(new Paragraph("第三列", fontChinese));
    table.addCell(cell3);
    // 写入流
    document.add(table);
    document.close();
} catch (Exception e) {
    log.error("export error! " + e);
}
````

+ 3 事件处理: PDFHeadFootEventHandler

````java
public class PDFHeadFootEventHandler extends PdfPageEventHelper {

    private Image image;
    private PdfTemplate total;
    private int presentFontSize = 8;
    private BaseFont bf = null;
    private Font fontDetail = null;

    public PDFHeadFootEventHandler() {

    }

    public PDFHeadFootEventHandler(Image image, int fontSize) {
        this.image = image;
        this.presentFontSize = fontSize;
    }

    @Override
    public void onOpenDocument(PdfWriter writer, Document document) {
        total = writer.getDirectContent().createTemplate(60, 60);
    }

    @Override
    public void onEndPage(PdfWriter writer, Document document) {
        try {
            if (bf == null) {
                bf = PDFUtil.getBaseFont(BaseFont.NOT_EMBEDDED);
            }
            if (fontDetail == null) {
                fontDetail = new Font(bf, presentFontSize, Font.NORMAL);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        // LOGO名称
        Phrase logoName = new Phrase("LOGO", fontDetail);
        ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_LEFT, logoName, document.left(),
            document.top() + 25, 0);

        // 批次信息
        Phrase batch = new Phrase("批次信息 ", fontDetail);
        ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_LEFT, batch, document.left(),
            document.top() + 18, 0);

        // 产品信息
        Phrase product = new Phrase("产品信息" , fontDetail);
        ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_LEFT, product, document.left(),
            document.top() + 11, 0);

        // 制表时间
        Phrase timeStr =
            new Phrase(String.format("开始时间:%s  截止时间:%s", "time start", "time end"), fontDetail);
        ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_RIGHT, timeStr, document.right(),
            document.top() + 11, 0);

        // 页眉下划线
        String underline = IntStream.range(0, 180).mapToObj(i -> "_").collect(Collectors.joining());
        ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_CENTER, new Phrase(underline, fontDetail),
            (document.rightMargin() + document.right() + document.leftMargin() - document.left()) / 2.0F,
            document.top() + 5, 0);

        // 页眉下划线
        String underlineFoot = IntStream.range(0, 180).mapToObj(i -> "_").collect(Collectors.joining());
        ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_CENTER,
            new Phrase(underlineFoot, fontDetail),
            (document.rightMargin() + document.right() + document.leftMargin() - document.left()) / 2.0F,
            document.bottom() - 5, 0);

        // 采集人
        if (request.getTimeInterval() != null) {
            Phrase timeInterval = new Phrase("采集人 ADMIN", fontDetail);
            ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_LEFT, timeInterval, document.left(),
                document.bottom() - 11, 0);
        }

        // 制表人
        Phrase createBy = new Phrase(String.format("制表人：%s", "ADMIN"), fontDetail);
        ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_RIGHT, createBy, document.right(),
            document.bottom() - 11, 0);

        // 制表时间
        Phrase createTime =
            new Phrase(String.format("制表时间：%s", DateUtil.format(new Date(), DateUtil.PATTERN_DATETIME)), fontDetail);
        ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_RIGHT, createTime, document.right(),
            document.bottom() - 18, 0);

        // 页码前半部分 第 X 页/共
        int pageNumber = writer.getPageNumber();
        String foot1 = String.format("第 %d 页 ", pageNumber);
        Phrase footer = new Phrase(foot1, fontDetail);

        // 计算 foot1 的长度
        float len = bf.getWidthPoint(foot1, presentFontSize);

        // 获取当前的 PdfContentByte
        PdfContentByte contentByte = writer.getDirectContent();

        ColumnText.showTextAligned(contentByte, Element.ALIGN_LEFT, footer,
            (document.rightMargin() + document.right() + document.leftMargin() - document.left() - len) / 2.0F - 10F,
            document.bottom() - 25, 0);

        // 添加 图片
        if (image != null) {
            try {
                document.add(image);
            } catch (DocumentException e) {
                e.printStackTrace();
            }
        }

        // 页码后半部分
        contentByte.addTemplate(total,
            (document.rightMargin() + document.right() + document.leftMargin() - document.left()) / 2.0F + 10F,
            document.bottom() - 25);

    }

    @Override
    public void onCloseDocument(PdfWriter writer, Document document) {
        // 页码后半部分
        total.beginText();
        total.setFontAndSize(bf, presentFontSize);
        String foot2 = String.format(" 共 %d 页", writer.getPageNumber());
        total.showText(foot2);
        total.endText();
        total.closePath();
    }

}
````
