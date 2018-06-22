---
layout: post
title: spring-boot项目使用tess4j识别文字
category: 技术
tags: PHP
description: spring-boot项目使用tess4j识别文字
---


1、引入maven

"<dependency>

		    <groupId>net.sourceforge.tess4j</groupId>
			
		    <artifactId>tess4j</artifactId>
			
		    <version>3.4.9</version>
			
</dependency>"

使用的tess4j版本是3.4.9，没有使用4.0之后的版本，但是使用的库文件是最新的4.0版本。

如果tess4j使用4..0的版本，则识别图片内容的时候，报错：read_params_file: parameter not found: enable_new_segsearch，

而且使用instance.setTessVariable("enable_new_segsearch", "0"),设置参数不起作用（其中instance为ITesseract instance = new Tesseract();）

2、添加目录和文件

在项目resource下添加目录名称必须为tessdata，在该目录下添加库文件，

文件名为：chi_sim.traineddata（中文简体字库）和eng.traineddata（英文字库），其中英文字库准确率高，若需要提高准确率，可以自己训练字库。

3、测试demo


import java.io.File;

import java.io.FileNotFoundException;

import org.springframework.util.ResourceUtils;

import net.sourceforge.tess4j.ITesseract;

import net.sourceforge.tess4j.Tesseract;

import net.sourceforge.tess4j.TesseractException;


public class OcrUtil {

	/**
	 * 从图片中提取文字,默认设置英文字库,使用classpath目录下的训练库
	 * 
	 * @param path
	 * @return
	 */
	public static String readChar(String path) {
		// JNA Interface Mapping
		ITesseract instance = new Tesseract();
		// JNA Direct Mapping
		// ITesseract instance = new Tesseract1();
		File imageFile = new File(path);
		try {
			File tessDataFolder = ResourceUtils.getFile("classpath:tessdata/");
			// 库地址
			instance.setDatapath(tessDataFolder.getAbsolutePath());
			// 语言,和库的名称相同
			instance.setLanguage("chi_sim");

			return getOCRText(instance, imageFile);
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
		return null;
	}

	/**
	 * 识别图片文件中的文字
	 * 
	 * @param instance
	 * @param imageFile
	 * @return
	 */
	private static String getOCRText(ITesseract instance, File imageFile) {
		String result = null;
		try {
			result = instance.doOCR(imageFile);
		} catch (TesseractException e) {
			e.printStackTrace();
		}
		return result;
	}

	public static void main(String[] args) {

		String path = "E:\\image\\0056.png";
		System.out.println(readChar(path));
	}

}


4、参考文档

http://tess4j.sourceforge.net/codesample.html

https://github.com/tesseract-ocr/tesseract/wiki/Data-Files#data-files-for-version-400-november-29-2016

https://blog.csdn.net/followwwind/article/details/78257340
