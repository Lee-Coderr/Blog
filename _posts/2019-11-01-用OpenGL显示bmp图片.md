---
layout: post
title: 用OpenGL显示bmp图片
date: 2019-11-01 13:20:00
Author: Lee-Coderr
categories: 
- OpenGL
- 数字图像处理
tags: [OpenGL]
comments: true
toc: true
pinned: true
---

要先掌握bmp文件的存储格式，读取图像信息后用 $glDrawPixels()$ 函数绘制。

直接上代码。

```c++
#include <windows.h>
#ifdef __APPLE__
#include <GLUT/glut.h>
#else
#include <GL/glut.h>
#endif
#include <bits/stdc++.h>
#include <stdlib.h>
using namespace std;
const double PI = acos(-1.0);

void init(){
    glClearColor(0.0, 0.0, 0.0, 0.0);
    glColor3f(1.0, 1.0, 1.0);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluOrtho2D(-1.0, 1.0, -1.0, 1.0);
}
#define GL_BGR_EXT 0x80E0
static GLint imagewidth;
static GLint imageheight;
static GLint pixellength;
static GLubyte* pixeldata;
void display(){
    glClear(GL_COLOR_BUFFER_BIT);
    glDrawPixels(imagewidth, imageheight, GL_BGR_EXT, GL_UNSIGNED_BYTE, pixeldata); //绘制像素
    glFlush();
    glutSwapBuffers();
}
int main(int argc, char **argv){

    FILE* pfile=fopen("D:\\img\\2.bmp","rb"); //打开文件
    if(pfile == 0) exit(0);

    fseek(pfile, 0x0012, SEEK_SET); //读取图像大小
    fread(&imagewidth,sizeof(imagewidth),1,pfile);
    fread(&imageheight,sizeof(imageheight),1,pfile);

    //cout << imagewidth << "***" << imageheight << endl;
    pixellength = imagewidth*3; //计算像素数据长度
    while(pixellength%4 != 0) pixellength++;
    pixellength *= imageheight;

    pixeldata = (GLubyte*)malloc(pixellength); //读取像素数据
    if(pixeldata == 0) exit(0);

    fseek(pfile, 54, SEEK_SET);
    fread(pixeldata, pixellength, 1, pfile);

    fclose(pfile); //关闭文件

    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
    glutInitWindowSize(imagewidth, imageheight);
    glutInitWindowPosition(0, 0);
    glutCreateWindow("用OpenGL显示bmp图片");
    glutDisplayFunc(display);
    init();
    glutMainLoop();
    free(pixeldata);
}

```

