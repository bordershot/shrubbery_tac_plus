node{
  stage('setup_env') {
    sh 'echo "FROM alanfranz/fwd-centos-7:latest" > Dockerfile'
    sh 'echo "RUN yum install -y gcc bison flex m4 pam-devel tcp-wrappers tcp_wrappers-devel" >> Dockerfile'
  }
  def buildcontainer = docker.build('fpm', '-f Dockerfile .').inside {
    stage('download') {
      git url: 'git@github.com:bordershot/shrubbery_tac_plus.git'
    }
    stage('build') {
      sh 'mkdir $WORKSPACE/fpm-build'
      sh 'if [ -d "dist" ]; then [ -d "dist/default" ] && rsync -a dist/default/ $OUTPUT_DIR; [ -d "dist/systemd" ] && rsync -a dist/systemd/ $OUTPUT_DIR; [ -d "dist/sysv" ] && rsync -a dist/sysv/ $OUTPUT_DIR; fi'
      sh './configure && make && make install DESTDIR=$WORKSPACE/fpm-build'
    }
    stage('package') {
      sh 'fpm -f -t rpm -s dir -n tacacs --version 1.0 --description "DD TacPlus" --depends pam-devel --depends tcp-wrappers -C $WORKSPACE/fpm-build .'
      sh 'rm -rf $WORKSPACE/fpm-build'
    }
  }
  stage('test') {
    docker.image('centos7', '-u 0').inside {
      sh 'yum -y install $WORKSPACE/tacacs*.rpm'
    }
  }
}
