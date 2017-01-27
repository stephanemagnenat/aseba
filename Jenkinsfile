#!groovy

// Jenkinsfile for compiling, testing, and packaging Aseba.
// Requires CMake plugin from https://github.com/davidjsherman/aseba-jenkins.git in global library.

// Ideally we should unarchive previously compiled Dashel and Enki libraries, but for
// now we conservatively recompile them.

pipeline {
	agent label:'' // use any available Jenkins agent

	// Jenkins will prompt for parameters when a branch is build manually
	// but will use default parameters when the entire project is built.
	parameters {
		stringParam(defaultValue: 'master', description: 'Dashel branch', name: 'branch_dashel')
		stringParam(defaultValue: 'master', description: 'Enki branch', name: 'branch_enki')
	}
	
	stages {
		stage('Prepare') {
			steps {
				echo "branch_dashel=${env.branch_dashel}"
				echo "branch_enki=${env.branch_enki}"

				// Dashel and Enki will be checked ou into externals/ directory.
				sh 'mkdir -p externals'
				dir('aseba') {
					checkout scm
					sh 'git submodule update --init'
				}
				dir('externals/dashel') {
					git branch: "${env.branch_dashel}", url: 'https://github.com/aseba-community/dashel.git'
				}
				dir('externals/enki') {
					git branch: "${env.branch_enki}", url: 'https://github.com/enki-community/enki.git'
				}
				stash excludes: '.git', includes: 'aseba/**,externals/**', name: 'source'
			}
		}
		// Everything will be built in build/ directory.
		// Everything will be installed in dist/ directory.
		stage('Dashel') {
			steps {
				parallel (
					"debian" : {
						node('debian') {
							// sh 'rm -rf build/* dist/*'
							unstash 'source'
							CMake([sourceDir: '$workDir/externals/dashel', label: 'debian', preloadScript: 'set -x',
								   buildDir: '$workDir/build/dashel/debian'])
							stash includes: 'dist/**', name: 'dist-dashel-debian'
						}
					},
					"macos" : {
						node('macos') {
							// sh 'rm -rf build/* dist/*'
							unstash 'source'
							CMake([sourceDir: '$workDir/externals/dashel', label: 'macos', preloadScript: 'set -x',
								   buildDir: '$workDir/build/dashel/macos'])
							stash includes: 'dist/**', name: 'dist-dashel-macos'
						}
					},
					"windows" : {
						node('windows') {
							// sh 'rm -rf build/* dist/*'
							unstash 'source'
							CMake([sourceDir: '$workDir/externals/dashel', label: 'windows', preloadScript: 'set -x',
								   buildDir: '$workDir/build/dashel/windows'])
							stash includes: 'dist/**', name: 'dist-dashel-windows'
						}
					}
				)
			}
		}
		stage('Enki') {
			steps {
				parallel (
					"debian" : {
						node('debian') {
							unstash 'source'
							script {
								env.debian_python = sh ( script: '''
python -c "import sys; print 'lib/python'+str(sys.version_info[0])+'.'+str(sys.version_info[1])+'/dist-packages'"
''', returnStdout: true).trim()
							}
							CMake([sourceDir: '$workDir/externals/enki', label: 'debian', preloadScript: 'set -x',
								   getCmakeArgs: "-DPYTHON_CUSTOM_TARGET:PATH=${env.debian_python}",
								   buildDir: '$workDir/build/enki/debian'])
							stash includes: 'dist/**', name: 'dist-enki-debian'
						}
					},
					"macos" : {
						node('macos') {
							unstash 'source'
							CMake([sourceDir: '$workDir/externals/enki', label: 'macos', preloadScript: 'set -x',
								   buildDir: '$workDir/build/enki/macos'])
							stash includes: 'dist/**', name: 'dist-enki-macos'
						}
					},
					"windows" : {
						node('windows') {
							unstash 'source'
							CMake([sourceDir: '$workDir/externals/enki', label: 'windows', preloadScript: 'set -x',
								   buildDir: '$workDir/build/enki/windows'])
							stash includes: 'dist/**', name: 'dist-enki-windows'
						}
					}
				)
			}
		}
		stage('Compile') {
			steps {
				parallel (
					"debian" : {
						node('debian') {
							unstash 'dist-dashel-debian'
							unstash 'dist-enki-debian'
							script {
								env.debian_dashel_DIR = sh ( script: 'dirname $(find $PWD/dist/debian -name dashelConfig.cmake | head -1)', returnStdout: true).trim()
								env.debian_enki_DIR = sh ( script: 'dirname $(find $PWD/dist/debian -name enkiConfig.cmake | head -1)', returnStdout: true).trim()
							}
							unstash 'source'
							CMake([sourceDir: '$workDir/aseba', label: 'debian', preloadScript: 'set -x',
								   envs: [ "dashel_DIR=${env.debian_dashel_DIR}", "enki_DIR=${env.debian_enki_DIR}" ] ])
							stash includes: 'dist/**', name: 'dist-aseba-debian'
						}
					},
					"macos" : {
						node('macos') {
							unstash 'dist-dashel-macos'
							unstash 'dist-enki-macos'
							script {
								env.macos_dashel_DIR = sh ( script: 'dirname $(find $PWD/dist/macos -name dashelConfig.cmake | head -1)', returnStdout: true).trim()
								env.macos_enki_DIR = sh ( script: 'dirname $(find $PWD/dist/macos -name enkiConfig.cmake | head -1)', returnStdout: true).trim()
							}
							echo "Found dashel_DIR=${env.dashel_DIR} enki_DIR=${env.enki_DIR}"
							unstash 'source'
							CMake([sourceDir: '$workDir/aseba', label: 'macos', preloadScript: 'set -x',
								   envs: [ "dashel_DIR=${env.macos_dashel_DIR}", "enki_DIR=${env.macos_enki_DIR}" ] ])
							stash includes: 'dist/**', name: 'dist-aseba-macos'
						}
					},
					"windows" : {
						node('windows') {
							unstash 'dist-dashel-windows'
							unstash 'dist-enki-windows'
							script {
								env.windows_dashel_DIR = sh ( script: 'dirname $(find $PWD/dist/windows -name dashelConfig.cmake | head -1)', returnStdout: true).trim()
								env.windows_enki_DIR = sh ( script: 'dirname $(find $PWD/dist/windows -name enkiConfig.cmake | head -1)', returnStdout: true).trim()
							}
							unstash 'source'
							CMake([sourceDir: '$workDir/aseba', label: 'windows', preloadScript: 'set -x',
								   envs: [ "dashel_DIR=${env.windows_dashel_DIR}", "enki_DIR=${env.windows_enki_DIR}" ] ])
							stash includes: 'dist/**', name: 'dist-aseba-windows'
						}
					}
				)
			}
		}
		stage('Test') {
			// Only do some tests. To do: add test labels in CMake to distinguish between
			// obligatory smoke tests (to be done for every PR) and extended tests only for
			// releases or for end-to-end testing.
			steps {
				node('debian') {
					unstash 'dist-aseba-debian'
					dir('build/debian') {
						sh "LANG=en_US.UTF-8 ctest -E 'e2e.*|simulate.*|.*http.*|valgrind.*'"
					}
				}
			}
		}
		stage('Extended Test') {
			// Extended tests are only run for master branch.
			when {
				true // env.BRANCH == 'master'
			}
			steps {
				node('debian') {
					unstash 'dist-aseba-debian'
					dir('build/aseba') {
						sh "LANG=en_US.UTF-8 ctest -R 'e2e.*|simulate.*|.*http.*|valgrind.*'"
					}
				}
			}
		}
		stage('Package') {
			// Packages are only built for master branch
			when {
				true // env.BRANCH == 'master'
			}
			steps {
				parallel (
					"debian" : {
						node('debian') {
							// NOT PORTABLE, relies on libdashel and libenki being installed on the node.
							// We should rebuild all packages in a clean environment using pbuilder.
							unstash 'dist-dashel-debian'
							unstash 'dist-enki-debian'
							unstash 'source'
							dir('aseba') {
								sh 'which debuild && debuild -i -us -uc -b'
							}
							archiveArtifacts artifacts: 'aseba*.deb', fingerprint: true, onlyIfSuccessful: true
							// sh 'mv aseba*.deb aseba*.changes aseba*.build dist/debian/'
							// stash includes: 'dist/**', name: 'package-debian'
						}
					}
				)
			}
		}
		stage('Archive') {
			steps {
				script {
					// Can't use collectEntries yet [JENKINS-26481]
					def p = [:]
					for (x in ['debian','macos','windows']) {
						def label = x
						p[label] = {
							node(label) {
								unstash 'dist-aseba-' + label
								archiveArtifacts artifacts: 'dist/**', fingerprint: true, onlyIfSuccessful: true
							}
						}
					}
					parallel p;
				}
			}
		}
	}
}
