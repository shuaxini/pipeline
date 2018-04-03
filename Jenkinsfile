#!/usr/bin/env groovy Jenkinsfile

pipeline {
 
	agent {
		node {
			label 'hddl_daily_build'
		    customWorkspace '/home/albert/xbintest_2'
		}
	}
	
    triggers {
        cron('H H/5 * * *')
    }
	
	options {
		timeout(time:12, unit:'MINUTES')
	}

    stages {
	
        stage('download'){
	        steps {
                script{
                    def resultCopy = sh 'cp -r /home/albert/xbinbp/python_test/*  .'
                    if (resultCopy == 0) {
                        echo 'cpoy failed'
                    }				    
					
					def resultDownload = sh './execute.py --user shouheng --pwd zsh05186! --modify "$manifest_args" -p hddl -b master --workspace $(pwd) --use_myx ON --build_type release'
					if (resultDownload == 0) {
						echo 'download failed'
					}
				}
			}
			post { 
				failure {
					emailext (
						subject: "'${env.JOB_NAME} [${env.BUILD_NUMBER}]' 下载失败",
						body: """
						详情：
						FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'             
						状态：${env.JOB_NAME} jenkins 下载失败 
						URL ：${env.BUILD_URL}
						项目名称 ：${env.JOB_NAME} 
						项目更新进度：${env.BUILD_NUMBER}
						""",
						to: "bin.xie@intel.com",
						recipientProviders: [[$class: 'DevelopersRecipientProvider']]
					)
				}
			}
        }
		
		stage('compile') {
			environment {
				def MV_TOOLS_DIR = "/home/albert/WORK/Tools/Releases/General"
				def MV_TOOLS_VERSION = "00.88.10"
				def DL_SDK_TEMP = "/home/albert/xbintemp"
				def HDDL_INSTALL_DIR = "/home/albert/xbintest_2/output/usr/local"
				def LD_LIBRARY_PATH = "${HDDL_INSTALL_DIR}/lib:${HDDL_INSTALL_DIR}/lib/ubuntu_16.04/intel64"
				def MFX_HOME="/opt/intel/mediasdk"
				
				def project_name = 'all'
			}
				
			steps {
				
				script{

					project_name = 'icv-fathom'
					sh 'make icv-fathom TYPE=release'

					project_name = 'inference-engine'
					sh 'make inference-engine TYPE=release'
					
					project_name = 'hddl-drivers'
					sh 'make hddl-drivers TYPE=release'
					
					project_name = 'hddl-bsl'
					sh 'make hddl-bsl TYPE=release'
					
					project_name = 'hddl-service-merged'
					sh 'make hddl-service-merged TYPE=release'
					
					project_name = 'hddl-samples'
					sh 'make hddl-samples TYPE=release'
					
				}
			}
			
			post { 
				failure {
					emailext (
						subject: "'${env.JOB_NAME} [${env.BUILD_NUMBER}]' $project_name 编译失败",
						body: """
						详情：
						FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'             
						状态：${env.JOB_NAME} jenkins $project_name 编译失败 
						URL ：${env.BUILD_URL}
						项目名称 ：${env.JOB_NAME} 
						项目更新进度：${env.BUILD_NUMBER}
						""",
						to: "bin.xie@intel.com",
						recipientProviders: [[$class: 'DevelopersRecipientProvider']]
					)
				}
			}
		}
        
		stage('unit test') {
			environment {
				def HDDL_INSTALL_DIR="/home/albert/daily_workspace/workspace/hddl_build_manual/output/usr/local"
				def LD_LIBRARY_PATH="${HDDL_INSTALL_DIR}/lib:$HDDL_INSTALL_DIR/lib/ubuntu_16.04/intel64"
				def HDDL_TESTCASE_DIR="/home/albert/hddl-testcase"
				def PATH="${PATH}:${HDDL_INSTALL_DIR}/bin"
				def HDDL_TEST_VERSION="V1"
			}
			
            steps {	
				dir('/home/albert/dev-validation') {
				    sh 'python mainT.py --config config/fullCycleConfig.xml --testCaseID BA15'
				    sh 'python mainT.py --config config/fullCycleConfig.xml --testCaseID BA16'
			    }				
				dir('/home/albert/hddl-testcase/mvnc/debug/myx_debug') {
					sh 'python testMYX.py'
				}
				
            }

			post { 
				failure {
					emailext (
						subject: "'${env.JOB_NAME} [${env.BUILD_NUMBER}]' 测试失败",
						body: """
						详情：
						FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'             
						状态：${env.JOB_NAME} jenkins 测试失败 
						URL ：${env.BUILD_URL}
						项目名称 ：${env.JOB_NAME} 
						项目更新进度：${env.BUILD_NUMBER}
						""",
						to: "bin.xie@intel.com",
						recipientProviders: [[$class: 'DevelopersRecipientProvider']]
					)
				}
			}
        }
    }
	
}
