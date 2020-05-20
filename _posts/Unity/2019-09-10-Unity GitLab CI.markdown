---
layout:     post
title:      "Unity GitLab CI"
subtitle:   " \"CI\""
date:       2019-09-10 17:00:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
    - Tool
---
* content
{:toc}

> 目前只有文件，没有说明,懂powershell的同学应该能看得明白！

## .gitlab-ci.yml
> ```
> stages:
>     - AssetBundle
>     - Export
>     - Build
> 
> before_script:
>   - git submodule sync --recursive
>   - git submodule update --init --recursive --force
> 
> BuildAssetBundle:
>     stage:
>         AssetBundle
>     when:
>         manual
>     tags:
>         - android
>     script:
>         - ./BeforeAssetBundleBuild.ps1
>         - Unity -batchmode -projectPath ${CI_PROJECT_DIR} -nographics > -executeMethod GitlabCI.BuildAssetsBundles -quit -logFile $> {CI_PROJECT_DIR}/Unity.Log -${ForceRebuild} -quit
>         - ./AfterAssetBundleBuild.ps1
>         
> Export_Gradle:
>     stage:
>         Export
>     when:
>         manual
>     tags:
>         - android
>     script:
>         - ./WeTest.ps1
>         - ./BeforeAssetBundleBuild.ps1
>         - Unity -batchmode -projectPath ${CI_PROJECT_DIR} -nographics > -executeMethod GitlabCI.ExportGradleProject -logFile $> {CI_PROJECT_DIR}/Unity.Log -quit -deployType:${deployType} -$> {RELEASE} -logLevel:${logLevel}
>         - ./AfterAssetBundleBuild.ps1
>     
> BuildApk:
>     stage: Build
>     when:
>         manual
>     tags:
>         - android
>     script:
>         - F:/android/gradle-2.10/bin/gradle.bat clean assembleRelease
>         - F:/android/gradle-2.10/bin/gradle.bat build
>         - cp ./build/outputs/apk/XXX-debug.apk ${CI_PROJECT_DIR}/AAA.apk
> 
>     artifacts:
>         expire_in: 1 day
>         paths:
>             - .\*.apk       
> ```

## BeforeAssetBundleBuild.ps1
> ```
> $scripterrorfilepath=${CI_PROJECT_DIR}+"/ScriptError.txt"
> 
> if (!(Test-Path ${scripterrorfilepath})) {
> 	New-Item ${scripterrorfilepath} -type file -force -value "C# compile > error!!!"
> }
> ```

## AfterAssetBundleBuild.ps1
> ```
> $nid = (Get-Process Unity).id
> 
> Wait-Process -Id $nid
> 
> $fileLog=${CI_PROJECT_DIR}+"/Unity.Log"
> 
> 
> $scripterrorfilepath=$CI_PROJECT_DIR+"/ScriptError.txt"
> 
> $errorfilepath=$CI_PROJECT_DIR+"/ExportAndroidError.txt"
> 
> $unitylog=Get-Content $fileLog
> echo $unitylog
> 
> # 遍历Compilation failed:字段
> $scriptErrorFlag="(Compilation failed: )"
> 
> if ($unitylog -match $scriptErrorFlag){
>     #$scriptErrorMatch="Compilation failed:"
>     #$f = select-string -path $fileLog -pattern $scriptErrorMatch > -context 0, 100
> 	#($f)[0].context | format-list
> 	#echo $f
> 	$flag=0
>     foreach ($n in $unitylog)
>     {
>         if( $n.StartsWith("Compilation failed:")){
>             $flag=1
>         }
>         if ($flag -eq 1){
>             echo $n
>         }
>         if ($n.StartsWith("-----EndCompilerOutput---------------")){
>             $flag=0
>         }
>     }
> }
> 
> if (Test-Path ${scripterrorfilepath}){
>     $res=Get-Content $scripterrorfilepath
>     echo $res
> 	exit 1
> }
> 
> if (Test-Path ${errorfilepath}){
>     $res=Get-Content $errorfilepath
>     echo $res
> 	exit 2
> }
> ```

## WeTest.ps1
> ```
> if ($WeTestEnable -eq "true" -and $RELEASE -ne "release"){
>     cp ${CI_PROJECT_DIR}/Tools/wetest/U3DAutomation.dll ${CI_PROJECT_DIR}/> Assets/Script/U3DAutomation.dll
> 
>     $smcsfilepath=${CI_PROJECT_DIR}+"/Assets/smcs.rsp"
> 
>     if (!(Test-Path ${smcsfilepath})) {
> 	    New-Item ${smcsfilepath} -type file -force -value > "-define:USE_WETEST"
>     }
>     cp ${CI_PROJECT_DIR}/Tools/wetest/U3dautomation.jar ${CI_PROJECT_DIR}/> Assets/Plugins/Android/U3dautomation.jar
>     cp ${CI_PROJECT_DIR}/Tools/wetest/libcrashmonitor.so ${CI_PROJECT_DIR}> /Assets/Plugins/Android/libcrashmonitor.so
> }
> ```

先把文件撸上去，以后再补上说明