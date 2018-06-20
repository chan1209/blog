---
layout: post
title: Jmeter学习笔记之2
tags:
  - Jmeter
  - ant
---

## Jmeter+ant触发

1. 环境问题

   * ant环境（自行百度）

   

2. ant的lib目录下引入Jmeter提供ant编译的jar包：Ant运行时才能找到"org.programmerplanet.ant.taskdefs.jmeter.JMeterTask"这个类，从而成功触发JMeter脚本 

    

3. 切换jmeter输出格式

   ```properties
   # legitimate values: xml, csv, db.  Only xml and csv are currently supported.
   jmeter.save.saveservice.output_format=xml
   ```

   

4. 主要思路

   运行jmx文件 

   -- > 生成.jtl文件 

   -- > ant使用指定的.xsl文件(build_smoke_report.xml定义) 解析.jtl文件

   -- > 生成html文件

   

5. build文件调整
   * jmeter根目录-- <property name="jmeter.home" value="D:\apache-jmeter-3.2" />

   * 脚本位置--<testplans dir="D:\apache-jmeter-3.2\jmeter_ant\sample" includes="*.jmx" />

   * 文件输出位置--

     <property name="jmeter.result.jtl.dir" value="D:\apache-jmeter-3.2\jmeter_ant\report" />

     <property name="jmeter.result.html.dir" value="D:\apache-jmeter-3.2\jmeter_ant\report" />

   * <xslt >  配置xml加载逻辑

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project name="ant-jmeter-test" default="run" basedir=".">
     	  <tstamp>
             <format property="time" pattern="_yyyy_MMdd_HHmmss" />
         </tstamp>
         <!-- 需要改成自己本地的 Jmeter 目录-->  
         <property name="jmeter.home" value="D:\apache-jmeter-3.2" />
         <property name="report.title" value="接口测试"/>
         <!-- jmeter生成jtl格式的结果报告的路径--> 
         <property name="jmeter.result.jtl.dir" value="D:\apache-jmeter-3.2\jmeter_ant\report" />
         <!-- jmeter生成html格式的结果报告的路径-->
         <property name="jmeter.result.html.dir" value="D:\apache-jmeter-3.2\jmeter_ant\report" />
         <property name="detail" value="_detail" />
         <!-- 生成的报告的前缀-->  
         <property name="ReportName" value="SmokeReport" />
         <property name="jmeter.result.jtlName" value="${jmeter.result.jtl.dir}/${ReportName}${time}.jtl" />
         <property name="jmeter.result.htmlName" value="${jmeter.result.html.dir}/${ReportName}${time}.html" />
     
         <target name="run">
             <antcall target="test" />
             <antcall target="report" />
         </target>
         
         <target name="test">
             <taskdef name="jmeter" classname="org.programmerplanet.ant.taskdefs.jmeter.JMeterTask" />
             <jmeter jmeterhome="${jmeter.home}" resultlog="${jmeter.result.jtlName}">
                 <!-- 声明要运行的脚本"*.jmx"指包含此目录下的所有jmeter脚本-->
                 <testplans dir="D:\apache-jmeter-3.2\jmeter_ant\sample" includes="*.jmx" />
                 
                 <property name="jmeter.save.saveservice.output_format" value="xml"/>
             </jmeter>
         </target>
             
         <path id="xslt.classpath">
             <fileset dir="${jmeter.home}/lib" includes="xalan*.jar"/>
             <fileset dir="${jmeter.home}/lib" includes="serializer*.jar"/>
         </path>
     
     
         <target name="report">
             <tstamp> <format property="report.datestamp" pattern="yyyy/MM/dd HH:mm" /></tstamp>
             <xslt 
                   classpathref="xslt.classpath"
                   force="true"
                   in="${jmeter.result.jtlName}"
                   out="${jmeter.result.htmlName}"
                   style="${jmeter.home}/extras/jmeter-results-detail-report_21.xsl">
                   <param name="dateReport" expression="${report.datestamp}"/>
            </xslt>
                     <!-- 因为上面生成报告的时候，不会将相关的图片也一起拷贝至目标目录，所以，需要手动拷贝 --> 
             <copy todir="${jmeter.result.html.dir}">
                 <fileset dir="${jmeter.home}/extras">
                     <include name="collapse.png" />
                     <include name="expand.png" />
                 </fileset>
             </copy>
         </target>
     
     </project>
     
     ```

     

     

6. ant执行

   * cmd进入build文件目录

   * ant -f %build_file_name%

     

7. xsl文件核心代码部分

   ps:前置操作

   * 将style文件放入jmeter.extras文件夹

   * jmeter.properties输出项配置调整

     ```properties
     jmeter.save.saveservice.data_type=true
     jmeter.save.saveservice.label=true
     jmeter.save.saveservice.response_code=true
     # response_data is not currently supported for CSV output
     jmeter.save.saveservice.response_data=true
     # Save ResponseData for failed samples
     jmeter.save.saveservice.response_data.on_error=false
     jmeter.save.saveservice.response_message=true
     jmeter.save.saveservice.successful=true
     jmeter.save.saveservice.thread_name=true
     jmeter.save.saveservice.time=true
     jmeter.save.saveservice.subresults=true
     jmeter.save.saveservice.assertions=true
     jmeter.save.saveservice.latency=true
     jmeter.save.saveservice.connect_time=true
     jmeter.save.saveservice.samplerData=true
     jmeter.save.saveservice.responseHeaders=true
     jmeter.save.saveservice.requestHeaders=true
     jmeter.save.saveservice.encoding=false
     jmeter.save.saveservice.bytes=true
     jmeter.save.saveservice.url=true
     jmeter.save.saveservice.filename=true
     jmeter.save.saveservice.hostname=true
     jmeter.save.saveservice.thread_counts=true
     jmeter.save.saveservice.sample_count=true
     jmeter.save.saveservice.idle_time=true
     ```

     

   获取测试数据部分

   ```xml
   <tr valign="top">
       <xsl:variable name="allCount" select="count(/testResults/*)" />
       <xsl:variable name="allFailureCount" select="count(/testResults/*[attribute::s='false'])" />
       <xsl:variable name="allSuccessCount" select="count(/testResults/*[attribute::s='true'])" />
       <xsl:variable name="allSuccessPercent" select="$allSuccessCount div $allCount" />
       <xsl:variable name="allTotalTime" select="sum(/testResults/*/@t)" />
       <xsl:variable name="allAverageTime" select="$allTotalTime div $allCount" />
       <xsl:variable name="allMinTime">
           <xsl:call-template name="min">
               <xsl:with-param name="nodes" select="/testResults/*/@t" />
           </xsl:call-template>
       </xsl:variable>
       <xsl:variable name="allMaxTime">
           <xsl:call-template name="max">
               <xsl:with-param name="nodes" select="/testResults/*/@t" />
           </xsl:call-template>
       </xsl:variable>
   	<xsl:variable name="allMedianLineTime">
           <xsl:call-template name="line">
               <xsl:with-param name="nodes" select="/testResults/*/@t" />
               <xsl:with-param name="position" select="floor($allCount * 0.5+1)" />
           </xsl:call-template>
       </xsl:variable>
       <xsl:variable name="allNinetyLineTime">
           <xsl:call-template name="line">
               <xsl:with-param name="nodes" select="/testResults/*/@t" />
               <xsl:with-param name="position" select="round($allCount * 0.1+1)" />  
           </xsl:call-template>
       </xsl:variable>
       <xsl:variable name="allNinetyFiveLineTime">
           <xsl:call-template name="line">
               <xsl:with-param name="nodes" select="/testResults/*/@t" />
               <xsl:with-param name="position" select="round($allCount * 0.05+1)" />
           </xsl:call-template>
       </xsl:variable>
       <xsl:variable name="allNinetyNineLineTime">
           <xsl:call-template name="line">
               <xsl:with-param name="nodes" select="/testResults/*/@t" />
               <xsl:with-param name="position" select="ceiling($allCount * 0.01)" />
           </xsl:call-template>
   	</xsl:variable>
       <xsl:variable name="allEndTime">
           <xsl:for-each select="/testResults/*/@ts">
               <xsl:sort data-type="number" order="descending"  />
               <xsl:if test="position() = 1">
                   <xsl:value-of select="number(.)+number(../@t)" />
               </xsl:if>
           </xsl:for-each>
       </xsl:variable>
       <xsl:variable name="allBeginTime">
           <xsl:for-each select="/testResults/*/@ts">
               <xsl:sort data-type="number" order="descending" />
               <xsl:if test="position() = $allCount">
                   <xsl:value-of select="number(.)" />
               </xsl:if>
           </xsl:for-each>
       </xsl:variable>
       <xsl:variable name="allNodeThroughput" select="$allCount div (number($allEndTime)-number($allBeginTime)) * 1000" />
       <xsl:variable name="allNodeKB" select="(sum(/testResults/*/@by) div 1024) div (number($allEndTime)-number($allBeginTime)) * 1000" />
       <xsl:attribute name="class">
           <xsl:choose>
               <xsl:when test="$allFailureCount &gt; 0">Failure</xsl:when>
           </xsl:choose>
       </xsl:attribute>
       <td align="center">
           <xsl:value-of select="$allCount" />
       </td>
       <td align="center">
           <xsl:value-of select="$allFailureCount" />
       </td>
       <td align="center">
           <xsl:call-template name="display-percent">
               <xsl:with-param name="value" select="$allSuccessPercent" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-time">
               <xsl:with-param name="value" select="$allAverageTime" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-time">
               <xsl:with-param name="value" select="$allMinTime" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-time">
               <xsl:with-param name="value" select="$allMaxTime" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-time">
               <xsl:with-param name="value" select="$allMedianLineTime" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-time">
               <xsl:with-param name="value" select="$allNinetyLineTime" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-time">
               <xsl:with-param name="value" select="$allNinetyFiveLineTime" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-time">
               <xsl:with-param name="value" select="$allNinetyNineLineTime" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-persecond">
               <xsl:with-param name="value" select="$allNodeThroughput" />
           </xsl:call-template>
       </td>
       <td align="center">
           <xsl:call-template name="display-decimal">
               <xsl:with-param name="value" select="$allNodeKB" />
           </xsl:call-template>
       </td>
    </tr>
   ```

   
