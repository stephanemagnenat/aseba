#!groovy

// Jenkinsfile for compiling, testing, and packaging Aseba.
// Requires CMake plugin from https://github.com/davidjsherman/aseba-jenkins.git in global library.

// Ideally we should unarchive previously compiled Dashel and Enki libraries, but for
// now we conservatively recompile them.

pipeline {
	agent label:'debian' // use any available Jenkins agent
	stages {
		stage('Prepare') {
			steps {
				sh 'mkdir -p build dist'
				dir('aseba') {
					checkout scm
					sh 'git submodule update --init'
				}
				dir('dashel') {
					git branch: 'pollsocketstream', url: 'https://github.com/davidjsherman/dashel.git'
				}
				dir('enki') {
					git branch: '602e3dd8cff678c27a3b0e4bc888637523b68707', url: 'https://github.com/davidjsherman/enki.git'
				}
				stash excludes: '.git', name: 'source'
			}
		}
		stage('Dashel') {
			steps {
				unstash 'source'
				CMake([buildType: 'Debug',
					   sourceDir: '$workDir/dashel',
					   buildDir: '$workDir/build/dashel',
					   installDir: '$workDir/dist',
					   getCmakeArgs: [ '-DBUILD_SHARED_LIBS:BOOL=ON' ]
					  ])
			}
			post {
				always {
					stash includes: 'dist/**', name: 'dashel'
				}
			}
		}
		stage('Enki') {
			steps {
				unstash 'source'
				CMake([buildType: 'Debug',
					   sourceDir: '$workDir/enki',
					   buildDir: '$workDir/build/enki',
					   installDir: '$workDir/dist',
					   getCmakeArgs: [ '-DBUILD_SHARED_LIBS:BOOL=ON' ]
					  ])
			}
			post {
				always {
					stash includes: 'dist/**', name: 'enki'
				}
			}
		}
		stage('Compile') {
			steps {
				unstash 'dashel'
				unstash 'enki'
				unstash 'source'
				script {
					env.dashel_DIR = sh ( script: 'dirname $(find dist -name dashelConfig.cmake | head -1)', returnStdout: true).trim()
				}
				CMake([buildType: 'Debug',
					   sourceDir: '$workDir/aseba',
					   buildDir: '$workDir/build/aseba',
					   installDir: '$workDir/dist',
					   envs: [ "dashel_DIR=${env.dashel_DIR}" ]
					  ])
			}
			post {
				always {
					stash includes: 'dist/**', name: 'aseba'
				}
			}
		}
		stage('Test') {
			steps {
				dir('build/aseba') {
					sh 'ctest'
				}
			}
		}
		stage('Package') {
			// when {
			// 	sh(script:'which debuild', returnStatus: true) == 0
			// }
			steps {
				unstash 'source'
				sh 'cd aseba && debuild -i -us -uc -b'
				sh 'mv aseba*.deb aseba*.changes aseba*.build dist/'
			}
		}
		stage('Archive') {
			steps {
				archiveArtifacts artifacts: 'dist/**', fingerprint: true, onlyIfSuccessful: true
			}
		}
	}
}
