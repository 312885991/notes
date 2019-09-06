## Excel表格的导入与导出

### 一、添加相关依赖

```dtd
<!-- http状态 -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.9</version>
</dependency>
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- 提供数据库访问 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
</dependency>
<!-- 处理Excel文件(必须) -->
 <dependency>
     <groupId>org.apache.poi</groupId>
     <artifactId>poi-ooxml</artifactId>
     <version>3.14</version>
 </dependency>
<!--添加Swagger依赖 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<!--添加Swagger-UI依赖 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```



### 二、配置文件

```xml-dtd
spring:

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/yonyou?serverTimezone=Asia/Shanghai&useSSL=false
    username: root
    password: 123456

  # JPA 配置
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect  #不加这句则默认为myisam引擎
    database: mysql

```



### 三、相关工具类

#### 3.1 ExcelType 类

```java
package com.yonyou.util;

public enum ExcelType {

    // 普通类型
    COMMON,

    // 日期类型
    DATE,

    // 时间类型
    TIME

}
```

#### 3.2、ExcelUtils 类

```java
package com.yonyou.util;

import org.apache.poi.hssf.usermodel.DVConstraint;
import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFDataValidation;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.hssf.util.HSSFColor;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.ss.util.CellRangeAddressList;
import org.apache.poi.xssf.usermodel.XSSFDataValidationConstraint;
import org.apache.poi.xssf.usermodel.XSSFDataValidationHelper;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.lang.reflect.Field;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.*;

/**
 * ExcelUtil工具类实现功能:
 * 导出时传入list<T>,即可实现导出为一个excel,其中每个对象Ｔ为Excel中的一条记录.
 * 导入时读取excel,得到的结果是一个list<T>.T是自己定义的对象.
 * 需要导出的实体对象只需简单配置注解就能实现灵活导出,通过注解您可以方便实现下面功能:
 * 1.实体属性配置了注解就能导出到excel中,每个属性都对应一列.
 * 2.列名称可以通过注解配置.
 * 3.导出到哪一列可以通过注解配置.
 * 4.鼠标移动到该列时提示信息可以通过注解配置.
 * 5.用注解设置只能下拉选择不能随意填写功能.
 * 6.用注解设置是否只导出标题而不导出内容,这在导出内容作为模板以供用户填写时比较实用.
 * 本工具类以后可能还会加功能,请关注我的博客: http://blog.csdn.net/lk_blog
 */
public class ExcelUtils<T> {
    Class<T> clazz;

    public ExcelUtils(Class<T> clazz) {
        this.clazz = clazz;
    }


    /**
     * 导入 Excel
     * @param sheetName 工作簿名称，不存在默认为第一个工作簿
     * @param input     文件输入流
     * @param excelVersion   excel 版本
     * @return
     */
    public List<T> importExcel(String sheetName, InputStream input, ExcelVersion excelVersion) {
        int maxCol = 0;
        List<T> list = new ArrayList<T>();
        try {

            Workbook workbook;

            switch (excelVersion) {
                case EXCEL_VERSION_03:
                    workbook = new HSSFWorkbook(input);
                    break;
                case EXCEL_VERSION_07:
                    workbook = new XSSFWorkbook(input);
                    break;
                default:
                    workbook = new HSSFWorkbook(input);
                    break;
            }

            Sheet sheet = workbook.getSheet(sheetName);

            if (!sheetName.trim().equals("")) {
                sheet = workbook.getSheet(sheetName);// 如果指定sheet名,则取指定sheet中的内容.
            }
            if (sheet == null) {
                sheet = workbook.getSheetAt(0); // 如果传入的sheet名不存在则默认指向第1个sheet.
            }
            int rows = sheet.getPhysicalNumberOfRows();

            if (rows > 0) {// 有数据时才处理
                // Field[] allFields = clazz.getDeclaredFields();// 得到类的所有field.
                List<Field> allFields = getMappedFiled(clazz, null);

                Map<Integer, Field> fieldsMap = new HashMap<Integer, Field>();// 定义一个map用于存放列的序号和field.
                for (Field field : allFields) {
                    // 将有注解的field存放到map中.
                    if (field.isAnnotationPresent(ExcelVOAttribute.class)) {
                        ExcelVOAttribute attr = field
                                .getAnnotation(ExcelVOAttribute.class);
                        int col = getExcelCol(attr.column());// 获得列号
                        maxCol = Math.max(col, maxCol);
                        field.setAccessible(true);// 设置类的私有字段属性可访问.
                        fieldsMap.put(col, field);
                    }
                }
                for (int i = 1; i < rows; i++) {// 从第2行开始取数据,默认第一行是表头.
                    Row row = sheet.getRow(i);
                    int cellNum = maxCol;
                    T entity = null;
                    for (int j = 0; j <= cellNum; j++) {
                        Cell cell = row.getCell(j);
                        if (cell == null) {
                            continue;
                        }
                        int cellType = cell.getCellType();
                        int dataFormat = cell.getCellStyle().getDataFormat();
                        String c = "";
                        if (cellType == HSSFCell.CELL_TYPE_BOOLEAN) {
                            c = String.valueOf(cell.getBooleanCellValue());
                        } else if (dataFormat == 20 || dataFormat == 32) {
                            SimpleDateFormat format = new SimpleDateFormat("HH:mm");
                            Date date = DateUtil.getJavaDate(Double.parseDouble(cell.getStringCellValue()));
                            c = format.format(date);
                        }else {
                            cell.setCellType(HSSFCell.CELL_TYPE_STRING);
                            c = cell.getStringCellValue();
                        }
                        if (c == null || c.equals("")) {
                            continue;
                        }
                        entity = (entity == null ? clazz.newInstance() : entity);// 如果不存在实例则新建.
                        Field field = fieldsMap.get(j);// 从map中得到对应列的field.
                        if (field == null) {
                            continue;
                        }
                        // 取得类型,并根据对象类型设置值.
                        Class<?> fieldType = field.getType();
                        cell2Field(entity, c, field, fieldType);

                    }
                    if (entity != null) {
                        list.add(entity);
                    }
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        } catch (NullPointerException e) {
            return list;
        }
        return list;
    }


    /**
     * 对list数据源将其里面的数据导入到excel表单
     *
     * @param sheetNames 工作表的名称
     * @param output     java输出流
     * @return
     */
    public boolean exportExcel(List<T> lists[], String sheetNames[],
                               OutputStream output, ExcelVersion excelVersion) {
        if (lists.length != sheetNames.length) {
            return false;
        }

        Workbook workbook;// 产生工作薄对象

        // 根据版本选择接口实现类
        switch (excelVersion) {
            case EXCEL_VERSION_03:
                workbook = new HSSFWorkbook();
                break;
            case EXCEL_VERSION_07:
                workbook = new XSSFWorkbook();
                break;
            default:
                workbook = new HSSFWorkbook();
                break;
        }


        for (int ii = 0; ii < lists.length; ii++) {
            List<T> list = lists[ii];
            String sheetName = sheetNames[ii];

            List<Field> fields = getMappedFiled(clazz, null);

            Sheet sheet = workbook.createSheet();// 产生工作表对象

            workbook.setSheetName(ii, sheetName);

            Row row;
            Cell cell;// 产生单元格
            CellStyle style = workbook.createCellStyle();
            style.setFillForegroundColor(HSSFColor.SKY_BLUE.index);
            style.setFillBackgroundColor(HSSFColor.GREY_40_PERCENT.index);
            row = sheet.createRow(0);// 产生一行
            // 写入各个字段的列头名称
            for (int i = 0; i < fields.size(); i++) {
                Field field = fields.get(i);
                ExcelVOAttribute attr = field
                        .getAnnotation(ExcelVOAttribute.class);
                int col = getExcelCol(attr.column());// 获得列号
                cell = row.createCell(col);// 创建列
                cell.setCellType(HSSFCell.CELL_TYPE_STRING);// 设置列中写入内容为String类型
                cell.setCellValue(attr.name());// 写入列名

                // 如果设置了提示信息则鼠标放上去提示.
                if (!attr.prompt().trim().equals("")) {
                    setPrompt(sheet, "", attr.prompt(), 1, 100, col, col,excelVersion);// 这里默认设了2-101列提示.
                }
                // 如果设置了combo属性则本列只能选择不能输入
                if (attr.combo().length > 0) {
                    setHSSFValidation(sheet, attr.combo(), 1, 100, col, col,excelVersion);// 这里默认设了2-101列只能选择不能输入.
                }
                cell.setCellStyle(style);
            }

            int startNo = 0;
            int endNo = list.size();
            // 写入各条记录,每条记录对应excel表中的一行
            for (int i = startNo; i < endNo; i++) {
                row = sheet.createRow(i + 1 - startNo);
                T vo = (T) list.get(i); // 得到导出对象.
                for (int j = 0; j < fields.size(); j++) {
                    Field field = fields.get(j);// 获得field.
                    field.setAccessible(true);// 设置实体类私有属性可访问
                    ExcelVOAttribute attr = field
                            .getAnnotation(ExcelVOAttribute.class);
                    try {
                        // 根据ExcelVOAttribute中设置情况决定是否导出,有些情况需要保持为空,希望用户填写这一列.
                        if (attr.isExport()) {
                            cell = row.createCell(getExcelCol(attr.column()));// 创建cell
                            cell.setCellType(HSSFCell.CELL_TYPE_STRING);
                            cell.setCellValue(field.get(vo) == null ? ""
                                    : String.valueOf(field.get(vo)));// 如果数据存在就填入,不存在填入空格.
                        }
                    } catch (IllegalArgumentException e) {
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }

        try {
            output.flush();
            workbook.write(output);
            output.close();
            return true;
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        } finally {
            if (workbook != null) {
                try {
                    workbook.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    public Workbook exportExcel(List<T> lists[], String sheetNames[], ExcelVersion excelVersion) {

        Workbook workbook;// 产生工作薄对象

        // 根据版本选择接口实现类
        switch (excelVersion) {
            case EXCEL_VERSION_03:
                workbook = new HSSFWorkbook();
                break;
            case EXCEL_VERSION_07:
                workbook = new XSSFWorkbook();
                break;
            default:
                workbook = new HSSFWorkbook();
                break;
        }


        for (int ii = 0; ii < lists.length; ii++) {
            List<T> list = lists[ii];
            String sheetName = sheetNames[ii];

            List<Field> fields = getMappedFiled(clazz, null);

            Sheet sheet = workbook.createSheet();// 产生工作表对象

            workbook.setSheetName(ii, sheetName);

            Row row;
            Cell cell;// 产生单元格
            CellStyle style = workbook.createCellStyle();
            style.setFillForegroundColor(HSSFColor.SKY_BLUE.index);
            style.setFillBackgroundColor(HSSFColor.GREY_40_PERCENT.index);
            row = sheet.createRow(0);// 产生一行
            // 写入各个字段的列头名称
            for (int i = 0; i < fields.size(); i++) {
                Field field = fields.get(i);
                ExcelVOAttribute attr = field
                        .getAnnotation(ExcelVOAttribute.class);
                int col = getExcelCol(attr.column());// 获得列号
                cell = row.createCell(col);// 创建列
                cell.setCellType(HSSFCell.CELL_TYPE_STRING);// 设置列中写入内容为String类型
                cell.setCellValue(attr.name());// 写入列名

                // 如果设置了提示信息则鼠标放上去提示.
                if (!attr.prompt().trim().equals("")) {
                    setPrompt(sheet, "", attr.prompt(), 1, 100, col, col,excelVersion);// 这里默认设了2-101列提示.
                }
                // 如果设置了combo属性则本列只能选择不能输入
                if (attr.combo().length > 0) {
                    setHSSFValidation(sheet, attr.combo(), 1, 100, col, col,excelVersion);// 这里默认设了2-101列只能选择不能输入.
                }
                cell.setCellStyle(style);
            }

            int startNo = 0;
            int endNo = list.size();
            // 写入各条记录,每条记录对应excel表中的一行
            for (int i = startNo; i < endNo; i++) {
                row = sheet.createRow(i + 1 - startNo);
                T vo = (T) list.get(i); // 得到导出对象.
                for (int j = 0; j < fields.size(); j++) {
                    Field field = fields.get(j);// 获得field.
                    Class<?> type = field.getType();
                    field.setAccessible(true);// 设置实体类私有属性可访问
                    ExcelVOAttribute attr = field
                            .getAnnotation(ExcelVOAttribute.class);
                    try {
                        // 根据ExcelVOAttribute中设置情况决定是否导出,有些情况需要保持为空,希望用户填写这一列.
                        if (attr.isExport()) {
                            cell = row.createCell(getExcelCol(attr.column()));// 创建cell
                            cell.setCellType(HSSFCell.CELL_TYPE_STRING);
                            Object o = field.get(vo);
                            //处理日期转成Excel表格时 带零问题 如：1998-02-04 0:0:0.0
                            if( field.getType() == Date.class){
                                if(o != null){
                                    try {
                                        Date date;
                                        //格式化日期形式
                                        date = new SimpleDateFormat("yyyy-MM-dd").parse(String.valueOf(o));
                                        String format = new SimpleDateFormat("yyyy-MM-dd").format(date);
                                        // 如果数据存在就填入,不存在填入空格.
                                        cell.setCellValue(format);
                                        break;

                                    } catch (ParseException e) {
                                        e.printStackTrace();
                                    }
                                }
                            }
                            // 如果数据存在就填入,不存在填入空格
                            cell.setCellValue(o == null ? "" : String.valueOf(o));
                        }
                    } catch (IllegalArgumentException e) {
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }

        return workbook;
    }

    /**
     * 对list数据源将其里面的数据导入到excel表单
     *
     * @param sheetName 工作表的名称
     *                  //     * @param sheetSize
     *                  每个sheet中数据的行数,此数值必须小于65536
     * @param output    java输出流
     */
    @SuppressWarnings("unchecked")
    public boolean exportExcel(List<T> list, String sheetName,
                               OutputStream output, ExcelVersion excelVersion) {
        //此处 对类型进行转换
        List<T> ilist = new ArrayList<>();
        for (T t : list) {
            ilist.add(t);
        }
        List<T>[] lists = new ArrayList[1];
        lists[0] = ilist;

        String[] sheetNames = new String[1];
        sheetNames[0] = sheetName;

        return exportExcel(lists, sheetNames, output,excelVersion);
    }

    public Workbook exportExcel(List<T> list, String sheetName, ExcelVersion excelVersion) {
        //此处 对类型进行转换
        List<T> ilist = new ArrayList<>();
        for (T t : list) {
            ilist.add(t);
        }
        List<T>[] lists = new ArrayList[1];
        lists[0] = ilist;

        String[] sheetNames = new String[1];
        sheetNames[0] = sheetName;
        return exportExcel(lists, sheetNames,excelVersion);
    }

    /**
     * 将EXCEL中A,B,C,D,E列映射成0,1,2,3
     *
     * @param col
     */
    public static int getExcelCol(String col) {
        col = col.toUpperCase();
        // 从-1开始计算,字母重1开始运算。这种总数下来算数正好相同。
        int count = -1;
        char[] cs = col.toCharArray();
        for (int i = 0; i < cs.length; i++) {
            count += (cs[i] - 64) * Math.pow(26, cs.length - 1 - i);
        }
        return count;
    }

    /**
     * 设置单元格上提示
     *
     * @param sheet         要设置的sheet.
     * @param promptTitle   标题
     * @param promptContent 内容
     * @param firstRow      开始行
     * @param endRow        结束行
     * @param firstCol      开始列
     * @param endCol        结束列
     * @return 设置好的sheet.
     */
    public static Sheet setPrompt(Sheet sheet, String promptTitle,
                                  String promptContent, int firstRow, int endRow, int firstCol,
                                  int endCol, ExcelVersion excelVersion) {
        // 构造constraint对象
        DataValidationConstraint constraint = DVConstraint
                .createCustomFormulaConstraint("DD1");


        // 四个参数分别是：起始行、终止行、起始列、终止列
        CellRangeAddressList regions = new CellRangeAddressList(firstRow,
                endRow, firstCol, endCol);
        // 数据有效性对象
        DataValidation data_validation_view;

        switch (excelVersion) {
            case EXCEL_VERSION_03:
                data_validation_view = new HSSFDataValidation(
                        regions, constraint);
                break;
            case EXCEL_VERSION_07:
                XSSFDataValidationHelper dvHelper = new XSSFDataValidationHelper((XSSFSheet) sheet);
                XSSFDataValidationConstraint dvConstraint = (XSSFDataValidationConstraint) dvHelper
                        .createCustomConstraint("DD1");
                data_validation_view = dvHelper.createValidation(
                        dvConstraint, regions);
                break;
            default:
                data_validation_view = new HSSFDataValidation(
                        regions, constraint);
                break;
        }

        // 插入提示信息
        data_validation_view.createPromptBox(promptTitle, promptContent);
        // 开启提示信息显示（03默认开启，07默认不开启）
        data_validation_view.setShowPromptBox(true);
        sheet.addValidationData(data_validation_view);
        return sheet;
    }

    /**
     * 设置某些列的值只能输入预制的数据,显示下拉框.
     *
     * @param sheet    要设置的sheet.
     * @param textlist 下拉框显示的内容
     * @param firstRow 开始行
     * @param endRow   结束行
     * @param firstCol 开始列
     * @param endCol   结束列
     * @return 设置好的sheet.
     */
    public static Sheet setHSSFValidation(Sheet sheet,
                                          String[] textlist, int firstRow, int endRow, int firstCol,
                                          int endCol, ExcelVersion excelVersion) {
        // 加载下拉列表内容
        DVConstraint constraint = DVConstraint
                .createExplicitListConstraint(textlist);
        // 设置数据有效性加载在哪个单元格上,四个参数分别是：起始行、终止行、起始列、终止列
        CellRangeAddressList regions = new CellRangeAddressList(firstRow,
                endRow, firstCol, endCol);
        // 数据有效性对象
        DataValidation data_validation_list;
        switch (excelVersion) {
            case EXCEL_VERSION_03:
                data_validation_list = new HSSFDataValidation(
                        regions, constraint);
                break;
            case EXCEL_VERSION_07:
                XSSFDataValidationHelper dvHelper = new XSSFDataValidationHelper((XSSFSheet) sheet);
                XSSFDataValidationConstraint dvConstraint = (XSSFDataValidationConstraint) dvHelper
                        .createExplicitListConstraint(textlist);
                data_validation_list = dvHelper.createValidation(
                        dvConstraint, regions);
                break;
            default:
                data_validation_list = new HSSFDataValidation(
                        regions, constraint);
                break;
        }
        sheet.addValidationData(data_validation_list);
        return sheet;
    }

    /**
     * 得到实体类所有通过注解映射了数据表的字段
     *
     * @return
     */
    @SuppressWarnings("rawtypes")
    private List<Field> getMappedFiled(Class clazz, List<Field> fields) {
        if (fields == null) {
            fields = new ArrayList<Field>();
        }

        Field[] allFields = clazz.getDeclaredFields();// 得到所有定义字段
        // 得到所有field并存放到一个list中.
        for (Field field : allFields) {
            if (field.isAnnotationPresent(ExcelVOAttribute.class)) {
                fields.add(field);
            }
        }
        if (clazz.getSuperclass() != null
                && !clazz.getSuperclass().equals(Object.class)) {
            getMappedFiled(clazz.getSuperclass(), fields);
        }

        return fields;
    }

    /**
     * 将单元格中的数据根据类型填入对象
     * @param entity
     * @param cell
     * @param field
     * @param fieldType
     * @throws IllegalAccessException
     */
    private void cell2Field(T entity, String cell, Field field, Class<?> fieldType) throws IllegalAccessException {
        if (String.class == fieldType) {
            // 如果是时间类型，转换成 HH:mm 格式
            if (field.getAnnotation(ExcelVOAttribute.class).type() == ExcelType.TIME) {
                SimpleDateFormat format = new SimpleDateFormat("HH:mm");
                Date date = DateUtil.getJavaDate(Double.parseDouble(cell));
                field.set(entity, format.format(date));
            } else {
                field.set(entity, String.valueOf(cell));
            }
        } else if ((Integer.TYPE == fieldType)
                || (Integer.class == fieldType)) {
            field.set(entity, Integer.parseInt(cell));
        } else if ((Long.TYPE == fieldType)
                || (Long.class == fieldType)) {
            // 如果是日期类型，格式化成时间戳
            if (field.getAnnotation(ExcelVOAttribute.class).type() == ExcelType.DATE) {
                field.set(entity, (Long.valueOf(cell) - 25569) * 86400);  //(Long.valueOf(c) - 70 * 365 - 19) * 86400)
            } else {
                field.set(entity, Long.valueOf(cell));
            }
        } else if ((Float.TYPE == fieldType)
                || (Float.class == fieldType)) {
            field.set(entity, Float.valueOf(cell));
        } else if ((Short.TYPE == fieldType)
                || (Short.class == fieldType)) {
            field.set(entity, Short.valueOf(cell));
        } else if ((Double.TYPE == fieldType)
                || (Double.class == fieldType)) {
            field.set(entity, Double.valueOf(cell));
        } else if ((Integer.TYPE == fieldType)
                || (Integer.class == fieldType)) {
            field.set(entity, Integer.valueOf(cell));
        }
        //处理日期特殊类型
        else if (Date.class == fieldType) {
            Date date = null;
            try{
                date = new SimpleDateFormat("yyyy-MM-dd").parse(cell);
            }catch (Exception e){
                System.out.println(e.getMessage());
            }
            field.set(entity, date);
        } else if (Character.TYPE == fieldType) {
            if ((cell != null) && (cell.length() > 0)) {
                field.set(entity, Character
                        .valueOf(cell.charAt(0)));
            }
        }
    }

    public enum ExcelVersion {
        // 03 版本
        EXCEL_VERSION_03,

        // 07 版本
        EXCEL_VERSION_07
    }
} 
```



#### 3.3、ExcelVOAttribute 类

```java
package com.yonyou.util;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target( { java.lang.annotation.ElementType.FIELD })
public @interface ExcelVOAttribute {

    /**
     * 导出到Excel中的名字. 
     */
    public abstract String name();

    /**
     * 配置列的名称,对应A,B,C,D.... 
     */
    public abstract String column();

    /**
     * 提示信息 
     */
    public abstract String prompt() default "";

    /**
     * 设置只能选择不能输入的列内容. 
     */
    public abstract String[] combo() default {};

    /**
     * 是否导出数据,应对需求:有时我们需要导出一份模板,这是标题需要但内容需要用户手工填写. 
     */
    public abstract boolean isExport() default true;


    /**
     * 特殊类型，比如时间等需要转换的类型
     */
    public abstract ExcelType type() default ExcelType.COMMON;
}
```



### 四、编写测试导入与导出的实体类

```java
package com.yonyou.entity;

import com.yonyou.util.ExcelType;
import com.yonyou.util.ExcelVOAttribute;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import javax.persistence.*;
import java.util.Date;

@Table(name = "user")
@Entity
@Data
@ApiModel(value = "用户信息")
/**
 * 测试Excel的导入与导出
 * @author 刘路生
 * @date 2019/9/5
 */
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @ApiModelProperty(name = "ID",example = "001")
    private Integer id;

    @Column(columnDefinition = "varchar(20) NOT NULL comment '用户姓名'")
    @ApiModelProperty(name = "用户名",example = "刘路生")
    @ExcelVOAttribute(name = "用户名", column = "A")
    private String name;

    @Column(columnDefinition = "char(12) NOT NULL comment '手机号'")
    @ApiModelProperty(name = "电话号码",example = "17770845325")
    @ExcelVOAttribute(name = "手机号码", column = "B")
    private String phone;

    @Column(columnDefinition = "Date NOT NULL comment '出生日期'")
    @ApiModelProperty(name = "出生日期",example = "2019-2-4")
    @ExcelVOAttribute(name = "出生日期", column = "D",type = ExcelType.DATE, prompt = "单元格选择文本类型填写,日期格式为yyyy-MM-dd形式")
    private Date birthdate;

    @Column(columnDefinition = "char(4) NOT NULL comment '性别'")
    @ApiModelProperty(name = "性别",example = "男")
    @ExcelVOAttribute(name = "性别",column = "C",combo = "男,女")
    private String sex;
}
```

### 五、编写相应的 Repository类 和 Service类

#### 5.1、UserRepository

```java
package com.yonyou.repository;

import com.yonyou.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Integer> {


}
```

#### 5.2、UserServiceImpl

```java
package com.yonyou.service.impl;

import com.yonyou.entity.User;
import com.yonyou.repository.UserRepository;
import com.yonyou.service.UserService;
import com.yonyou.vo.UserVO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    /**
     * 查出ID为1的用户信息
     * @return
     */
    @Override
    public User getExampleUser() {
        //User user = userRepository.getOne(1);
        Optional<User> byId = userRepository.findById(1);
        if( byId.isPresent() ){
            User user = byId.get();
            return user;
        }
        return null;
    }

    /**
     * 存储用户信息
     * @param user
     * @return
     */
    @Override
    public User saveUser(User user) {
        return userRepository.save(user);
    }

    /**
     * 查询全部信息
     * @return
     */
    @Override
    public List<User> listUser() {
        return userRepository.findAll();
    }
}
```

### 六、编写 Controller 类

#### 6.1、UserController

```java
package com.yonyou.controller;


import com.yonyou.entity.User;
import com.yonyou.result.Result;
import com.yonyou.service.UserService;
import com.yonyou.util.ExcelUtils;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.apache.poi.ss.usermodel.Workbook;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.List;

@Api(tags = "Excel文件的导入与导出")
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    /**
     * 导入Excel
     * @param file
     * @return
     */
    @ApiOperation(value = "导入Excel表格", notes = "导入Excel表格")
    @PostMapping("/excelImport")
    public Result excelImport(@RequestParam("file")MultipartFile file){
        if(file.isEmpty()){
            return Result.error("导入文件为空！");
        }
        //获取上传的文件名
        String fileName = file.getOriginalFilename();
        if(fileName == null || fileName.trim().isEmpty()){
            return Result.error("导入的文件名不能为空");
        }
        //获取文件类型
        String fileType = "";
        fileType = fileName.substring(fileName.lastIndexOf(".")+1);
        String allowType = "xlsx,xls";
        if(allowType.indexOf(fileType.toLowerCase()) == -1){
            return Result.error("上传的文件不是Excel文件！");
        }
        try{
            InputStream in = file.getInputStream();
            List<User> users = new ExcelUtils<>(User.class).importExcel("Excel模板", in, ExcelUtils.ExcelVersion.EXCEL_VERSION_07);
            int start =0;
            boolean flag =false;
            System.out.println("导入的Excel文件中共包含"+ users.size()+"条数据");
            for(User user : users){
                start++;
                try{
                    System.out.println("正在导入第"+ start +"条数据");
                    userService.saveUser(user);
                    System.out.println("第"+ start +"条数据导入成功");
                    flag = true;
                }catch (Exception e){
                    System.out.println("第"+ start +"条数据导入失败");
                    System.out.println(e.getMessage());
                }
            }
            if(flag){
                return Result.success();
            }
            return Result.error("导入失败");
        }catch (Exception e){
            e.printStackTrace();
        }
        return Result.success();
    }

    /**
     * 导出Excel(下载模板)
     * @return
     */
    @ApiOperation(value = "下载模板", notes = "导出Excel表格")
    @GetMapping("/excelExport")
    public void excelExport(HttpServletResponse response)throws Exception{
        String fileName = "Excel模板";
        //List<User> list = userService.listUser();
        User exampleUser = userService.getExampleUser();
        List<User> list = new ArrayList<>();
        if(exampleUser != null){
            list.add(exampleUser);
        }
        Workbook workbook = new ExcelUtils<>(User.class).exportExcel(list, fileName, ExcelUtils.ExcelVersion.EXCEL_VERSION_07);
        setResponseHeader(response, fileName+".xlsx");
        //获取响应输出流
        ServletOutputStream out = response.getOutputStream();
        workbook.write(out);
        out.flush();
        out.close();
    }

    /**
     * 设置下载文件的响应头
     * @param response
     * @param fileName
     */
    public void setResponseHeader(HttpServletResponse response, String fileName) {
        try {
            fileName = new String(fileName.getBytes(), "ISO8859-1");
            response.setContentType("application/force-download;charset=ISO8859-1");
            response.addHeader("Content-Disposition", "attachment;fileName=" + fileName);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
}
```

#### 6.2、结果

![](t1.png)

##### 6.2.1、导出结果

![](t2.png)

![](t3.png)

##### 6.2.2、导入结果

![](t4.png)



### * 其中用到的处理结果的 Result 类

```java
package com.yonyou.result;

import lombok.Data;

import org.apache.http.HttpStatus;

import java.io.Serializable;
import java.util.HashMap;

/**
 * @Package com.mylnet.purple.common.result
 * @ClassName Result.java
 * @Version 1.0.0
 * @Description TODO 相应结果统一封装
 * @Date: 2019/8/12 13:26
 */
@Data
public class Result<T> extends HashMap<String, Object> implements Serializable {
	
	/**
	 *  成功时候的调用
	 * */
	public static <T> Result<T> success(T data){
		return new Result<T>(data);
	}

	/**
	 *  成功时候的调用
	 * */
	public static <T> Result<T> success(){
		return new Result<T>();
	}

	@Override
	public Result put(String key, Object value) {
		super.put(key, value);
		return this;
	}

	
	/**
	 *  失败时候的调用
	 * */
	public static <T> Result<T> error(CodeMsg codeMsg){
		return new Result<T>(codeMsg);
	}

	public static Result error() {
		return error(HttpStatus.SC_INTERNAL_SERVER_ERROR, "未知异常，请联系管理员");
	}

	public static Result unAuthorized() {
		return error(HttpStatus.SC_UNAUTHORIZED, "");
	}

	public static Result error(String msg) {
		return error(HttpStatus.SC_INTERNAL_SERVER_ERROR, msg);
	}

	public static Result error(int code, String msg) {
		return new Result(code, msg, false);
	}

	public Result(){
		put("code", 200);
		put("flag", true);
		put("msg", "success");
	}

	private Result(T data) {
		put("code", 200);
		put("msg", "success");
		put("flag", true);
		put("data", data);
	}
	
	private Result(int code, String msg) {
		put("code", code);
		put("msg", msg);
	}

	private Result(int code, String msg, Boolean flag) {
		put("code", code);
		put("msg", msg);
		put("flag", flag);
	}
	
	private Result(CodeMsg codeMsg) {
		if(codeMsg != null) {
			put("code", codeMsg.getCode());
			put("msg", codeMsg.getMsg());
			put("flag", false);
		}
	}

}
```

### * 可能用到的 CodeMsg 类

```java
package com.yonyou.result;

import java.io.Serializable;

/**
 * @Package com.mylnet.purple.common.result
 * @ClassName CodeMsg.java
 * @Version 1.0.0
 * @Description TODO 通用错误码
 * @Date: 2019/8/12 13:26
 */
public class CodeMsg implements Serializable {
    private int code;
	private String msg;

	// 通用的错误码
	public static CodeMsg SUCCESS = new CodeMsg(0, "success");
	public static CodeMsg ERROR = new CodeMsg(1, "error");
	public static CodeMsg SERVER_ERROR = new CodeMsg(500100, "服务端异常");
	public static CodeMsg BIND_ERROR = new CodeMsg(500101, "参数校验异常：%s");

	// 认证授权相关
	public static CodeMsg UNAUTHORIZED = new CodeMsg(401100, "用户无权限");
	public static CodeMsg TOKEN_IS_EMPTY = new CodeMsg(401101, "token为空");
	public static CodeMsg TOKEN_IS_INVALID = new CodeMsg(401102, "token无效");

	public static CodeMsg FORBIDDEN_REQUEST = new CodeMsg(403100, "请求被拒绝");


	// 系统模块
	public static CodeMsg SYS_USER_NOT_EXIST = new CodeMsg(500201, "用户不存在");
	public static CodeMsg SYS_USERNAME_DUPLICATED = new CodeMsg(500202, "用户名重复");
    public static CodeMsg SYS_USER_PSW_ERROR = new CodeMsg(500203, "原密码不正确");
    public static CodeMsg SYS_USER_PSW_UPDATE_CANCEL = new CodeMsg(500204, "新密码和原来密码相同，无需更新");

    //app模块
	public static CodeMsg USER_NOT_EXIST_OR_PHONE_ERROR = new CodeMsg(500205, "用户不存在或手机号有误");
	public static CodeMsg PHONE_EXIST = new CodeMsg(500206, "该手机号已被注册");
	public static CodeMsg PHONE_ERROR = new CodeMsg(500207, "手机号不合法");

    public static CodeMsg SYS_PRAMS_VALUE_DUPLICATED = new CodeMsg(500208, "参数值重复");
    public static CodeMsg SYS_RECORDS_NOT_EXIST = new CodeMsg(500209, "不存在该记录");
	public static CodeMsg PHONE_NOT_EXIST = new CodeMsg(500302, "手机号不存在");
	public static CodeMsg CODE_ERROR = new CodeMsg(500303, "验证码错误");
	public static CodeMsg LACK_PARAMETER = new CodeMsg(500304, "缺少请求参数");
	public static CodeMsg SEND_ERROR = new CodeMsg(500305, "验证码发送失败");

	// 单据模块 (示例)
	public static CodeMsg BILL_XXX = new CodeMsg(500301, "单据异常");

	private CodeMsg( ) {

	}
			
	private CodeMsg( int code,String msg ) {
		this.code = code;
		this.msg = msg;
	}
	
	public int getCode() {
		return code;
	}
	public void setCode(int code) {
		this.code = code;
	}
	public String getMsg() {
		return msg;
	}
	public void setMsg(String msg) {
		this.msg = msg;
	}
	
	public CodeMsg fillArgs(Object... args) {
		int code = this.code;
		String message = String.format(this.msg, args);
		return new CodeMsg(code, message);
	}

	@Override
	public String toString() {
		return "CodeMsg [code=" + code + ", msg=" + msg + "]";
	}
	
}
```

