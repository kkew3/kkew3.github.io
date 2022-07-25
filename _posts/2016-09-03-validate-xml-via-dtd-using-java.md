---
layout: post
title:  "使用Java API通过DTD方式验证XML"
date:   2016-09-03 00:26:30 +0800
categories: java
---

# 摘要

本文记述了如何使用`Java 8`API 解析但不验证、按照XML文件头的`DOCTYPE`声明验证、使用本地文件验证XML的方法。本文不涉及如何读取、修改XML节点，以及创建XML文档的内容。

# 解析但不验证

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

import org.w3c.dom.Document;
import org.xml.sax.SAXException;

public class XMLParser {
	public static void main(String[] args) {
		try {
			String xmlToParse = "myDocument.xml";
			DocumentBuilderFactory dbf = 
					DocumentBuilderFactory.newInstance();
			// 默认DocumentBuilderFactory不创建
			// 启用验证功能的DocumentBuilder
			DocumentBuilder db = dbf.newDocumentBuilder();
			Document myDoc = db.parse(xmlToParse);
		} catch (ParseConfigurationException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (SAXException e) {
			e.printStackTrace();
		}
	}
```

# 使用XML文件头部声明的DOCTYPE验证

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

import org.w3c.dom.Document;
import org.xml.sax.SAXException;

public class XMLParser {
	public static void main(String[] args) {
		try {
			String xmlToParse = "myDocument.xml";
			DocumentBuilderFactory dbf = 
					DocumentBuilderFactory.newInstance();
			dbf.setValidating(true);  // 注意这里不同
			DocumentBuilder db = dbf.newDocumentBuilder();
			Document myDoc = db.parse(xmlToParse);
		} catch (ParseConfigurationException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (SAXException e) {
			e.printStackTrace();
		}
	}
```

这时可能抛出`IOException`，原因通常是没有找到XML所声明的DTD文件

* 如果XML声明的DTD在本地，可能会报`FileNotFoundException`。此时需要检查本地DTD的路径是否填写正确
* 否则可能报`SocketException`。此时需要检查网络是否畅通

**然而此时即使XML不符合所声明DTD的定义，`SAXException`也可能不会被抛出**，而仅仅是报错信息通过`System.err`打印出来，同时会打印运行警告：“警告: 已启用验证, 但未设置 org.xml.sax.ErrorHandler, 这可能不是预期结果。解析器将使用默认 ErrorHandler 来输出前 0 个错误。请调用 'setErrorHandler' 方法以解决此问题。”

这是因为没有设置`ErrorHandler`。如果希望`SAXException`在发生验证错误时被抛出，需要通过`DocumentBuilder.setErrorHandler(ErrorHandler eh)`方法进行设置。

重写上述代码如下：

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

import org.w3c.dom.Document;
import org.xml.sax.ErrorHandler;  // 注意这里不同
import org.xml.sax.SAXException;
import org.xml.sax.SAXParseException;  // 注意这里不同

public class XMLParser {
	public static void main(String[] args) {
		try {
			String xmlToParse = "myDocument.xml";
			DocumentBuilderFactory dbf = 
					DocumentBuilderFactory.newInstance();
			dbf.setValidating(true);
			DocumentBuilder db = dbf.newDocumentBuilder();
			db.setErrorHandler(new ErrorHandler() {
				/*
				 * 定义了一个只要出一点解析错误就抛出异常的ErrorHandler。
				 * 读者可以以此为依据编写更精细化管理的ErrorHandler。
				 */
				
				@Override
				public void error(SAXParseException exception)
						throws SAXException {
					throw exception;
				}
				@Override
				public void fatalError(SAXParseException exception)
						throws SAXException {
					throw exception;
				}
				@Override
				public void warning(SAXParseException exception)
						throws SAXException {
					throw exception;
				}
			});  // 注意这里不同
			Document myDoc = db.parse(xmlToParse);
		} catch (ParseConfigurationException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (SAXException e) {
			e.printStackTrace();
		}
	}
```

# 使用本地DTD文件验证

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

import org.w3c.dom.Document;
import org.xml.sax.ErrorHandler;
import org.xml.sax.EntityHandler;  // 注意这里不同
import org.xml.sax.InputSource;  // 注意这里不同
import org.xml.sax.SAXException;
import org.xml.sax.SAXParseException;

public class XMLParser {
	public static void main(String[] args) {
		try {
			String xmlToParse = "myDocument.xml";
			DocumentBuilderFactory dbf = 
					DocumentBuilderFactory.newInstance();
			dbf.setValidating(true);
			DocumentBuilder db = dbf.newDocumentBuilder();
			db.setErrorHandler(new ErrorHandler() {
				@Override
				public void error(SAXParseException exception)
						throws SAXException {
					throw exception;
				}
				@Override
				public void fatalError(SAXParseException exception)
						throws SAXException {
					throw exception;
				}
				@Override
				public void warning(SAXParseException exception)
						throws SAXException {
					throw exception;
				}
			});
			db.setEntityResolver(new EntityResolver() {
				/*
				 * 编写了根据PUBLIC域使用相应的本地dtd的EntityResolver；
				 * 读者也可以据此编写根据SYSTEM域使用相应dtd的EntityResolver；
				 * 或不管xml中声明成什么DOCTYPE，都使用同一份dtd进行验证，
				 * 此时resolveEntity方法体中仅包含
				 *     return new InputSource("a-fixed-dtd-path");
				 */
				
				@Override
				public InputSource resolveEntity(String publicId,
						String systemId) {
					switch (publicId) {  // 此处仅为示意
					case "URL-sample-1":
						return new InputSource(
								"local-dtd-path-for-url-sample-1");
					case "URL-sample-2":
						return new InputSource(
								"local-dtd-path-for-url-sample-2");
					default:
						// 仍然按照DOCTYPE去解析，此时可能抛出IOException
						return null;
				}
			});  // 注意这里不同
			Document myDoc = db.parse(xmlToParse);
		} catch (ParseConfigurationException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (SAXException e) {
			e.printStackTrace();
		}
	}
```
