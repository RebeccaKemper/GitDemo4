#!groovy 
2  
 
3  import com.sap.piper.Utils 
4  import com.sap.piper.ConfigurationLoader 
5  import com.sap.piper.ConfigurationMerger 
6  
 
7  /** 
8   *	Copyright (c) 2017 SAP SE or an SAP affiliate company.  All rights reserved. 
9   * 
10   *	This software is the confidential and proprietary information of SAP 
11   *	("Confidential Information"). You shall not disclose such Confidential 
12   *	Information and shall use it only in accordance with the terms of the 
13   *	license agreement you entered into with SAP. 
14  */ 
15  
 
16  @Library('piper-library-os') _ 
17  
 
18  CONFIG_FILE_PROPERTIES = '.pipeline/config.properties' 
19  CONFIG_FILE_YML = '.pipeline/config.yml' 
20  
 
21  node() { 
22    //Global variables: 
23    APP_PATH = 'src' 
24    SRC = "${WORKSPACE}/${APP_PATH}" 
25  
 
26    def CONFIG_FILE 
27  
 
28    def STEP_CONFIG_NEO_DEPLOY='neoDeploy' 
29    def STEP_CONFIG_MTA_BUILD='mtaBuild' 
30  
 
31    stage("Clone sources and setup environment"){ 
32      deleteDir() 
33      Map neoDeployConfiguration, mtaBuildConfiguration 
34      dir(APP_PATH) { 
35        checkout scm 
36        if(fileExists(CONFIG_FILE_YML) ) { 
37            CONFIG_FILE = CONFIG_FILE_YML 
38        } else if(fileExists (CONFIG_FILE_PROPERTIES) ) { 
39            CONFIG_FILE = CONFIG_FILE_PROPERTIES 
40        } else { 
41            error "No config file found." 
42        } 
43        echo "[INFO] using configuration file '${CONFIG_FILE}'." 
44        setupCommonPipelineEnvironment script: this, configFile: CONFIG_FILE 
45        prepareDefaultValues script: this 
46        neoDeployConfiguration = ConfigurationMerger.merge([:], (Set)[], 
47                                                           ConfigurationLoader.stepConfiguration(this, STEP_CONFIG_NEO_DEPLOY), (Set)['neoHome', 'account'], 
48                                                           ConfigurationLoader.defaultStepConfiguration(this, 'neoDeploy')) 
49        mtaBuildConfiguration = ConfigurationMerger.merge([:], (Set)[], 
50                                                          ConfigurationLoader.stepConfiguration(this, STEP_CONFIG_MTA_BUILD), (Set)['mtaJarLocation'], 
51                                                          ConfigurationLoader.defaultStepConfiguration(this, 'mtaBuild')) 
52      } 
53      MTA_JAR_LOCATION = mtaBuildConfiguration.mtaJarLocation ?: commonPipelineEnvironment.getConfigProperty('MTA_HOME') 
54      NEO_HOME = neoDeployConfiguration.neoHome ?: commonPipelineEnvironment.getConfigProperty('NEO_HOME') 
55      proxy = commonPipelineEnvironment.getConfigProperty('proxy') ?: '' 
56      httpsProxy = commonPipelineEnvironment.getConfigProperty('httpsProxy') ?: '' 
57    } 
58  
 
59    stage("Build Fiori App"){ 
60      dir(SRC){ 
61        withEnv(["http_proxy=${proxy}", "https_proxy=${httpsProxy}"]) { 
62          MTAR_FILE_PATH = mtaBuild script: this, mtaJarLocation: MTA_JAR_LOCATION, buildTarget: 'NEO' 
63        } 
64      } 
65    } 
66     
67    stage("Deploy Fiori App"){ 
68      dir(SRC){ 
69        withEnv(["http_proxy=${proxy}", "https_proxy=${httpsProxy}"]) { 
70          neoDeploy script: this, neoHome: NEO_HOME, archivePath: MTAR_FILE_PATH 
71        } 
72      } 
73    } 
74  } 
75  
 
76  
 
