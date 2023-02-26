# AWS GUI Flows with Vivado IPI

## Table of Content

1. [Overview](#overview)

2. [IP Integrator Project with Example Design](#ipiprojex)

3. [RTL Project with Example Design](#rtlprojex)

4. [RTL Project - New RTL Design](#rtlnew)



<a name="overview"></a>
# Overview  

This document covers top level steps (cheat sheet) for using a particular flow


<a name="ipiprojex"></a>
# IP Integrator Project with Example Design

Create project directory

start vivado

## Execute example
가능한 예제를 확인하려면 TCL 콘솔에 입력하세요.

aws::make\_ipi -examples

필요한 예제에 따라 TCL 콘솔에 입력합니다.

aws::make\_ipi -examples <example requested>

## Implementing the Design/Tar file

디자인 실행 탭에서 impl\_1을 마우스 오른쪽 버튼으로 클릭하고 실행 시작...을 선택합니다. 실행 시작 대화 상자에서 확인을 클릭합니다.  누락된 합성 결과 대화 상자에서 확인을 클릭합니다.

그러면 합성과 구현이 모두 실행됩니다.

완성된 .tar 파일은 <project>.runs/faas\_1/build/checkpoints/to\_aws/<timestamp>.Developer\_CL.tar에 있습니다. 설계에서 .tar를 사용하여 AFI/GAFI를 생성하는 방법에 대한 자세한 내용은 CL 예제 중 하나에서 Amazon FPGA 이미지(AFI)를 생성하는 방법을 참조하세요: 단계별 가이드 문서를 참조하십시오.

<a name="rtlprojex"></a>
# RTL Project with Example Design

Create project directory

start vivado

## Execute example
See what examples are possible, type into TCL console.

aws::make\_rtl -examples

Type into TCL console based upon example needed.

aws::make\_rtl -examples <example requested>

## Implementing the Design/Tar file

디자인 실행 탭에서 impl\_1을 마우스 오른쪽 버튼으로 클릭하고 실행 시작...을 선택합니다. 실행 시작 대화 상자에서 확인을 클릭합니다.  누락된 합성 결과 대화 상자에서 확인을 클릭합니다.

그러면 합성과 구현이 모두 실행됩니다.

완성된 .tar 파일은 <project>.runs/faas\_1/build/checkpoints/to\_aws/<timestamp>.Developer\_CL.tar에 있습니다. 설계에서 .tar를 사용하여 AFI/GAFI를 생성하는 방법에 대한 자세한 내용은 CL 예제 중 하나에서 Amazon FPGA 이미지(AFI)를 생성하는 방법을 참조하세요: 단계별 가이드 문서를 참조하십시오.

<a name="rtlnew"></a>
# RTL Project - New RTL Design

프로젝트 디렉토리 만들기

비바도 시작

## Create Vivado Project
aws::make\_rtl

RTL 소스, 시뮬레이션 소스, 컨스트레인트를 추가합니다.



