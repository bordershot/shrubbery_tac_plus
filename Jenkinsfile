def osmap = ['Ubuntu1604':['packaging':'deb', 'init':'systemd', 'build_env:'', 'build_deps':'', 'install_deps':[], test_env:[]], 
             'Ubuntu1404':'Ubuntu-1404-server/Ubuntu-1404-server0-000002.vmdk', 
             'Fedora24':'Fedora-24/Fedora-240-000001.vmdk', 
             'CentOS7':['packaging':'rpm', 'init':'systemd', 'build_env:'fwd-centos-7:latest', 
                        'build_deps':'gcc bison flex m4 pam-devel tcp_wrappers tcp_wrappers-devel', 
                        'install_deps':['pam-devel', 'tcp_wrappers'], test_env:'centos:7']
            ]
node{
  stage('setup_env') {
    sh 'echo "FROM alanfranz/fwd-centos-7:latest" > Dockerfile'
    sh 'echo "RUN yum install -y gcc bison flex m4 pam-devel tcp_wrappers tcp_wrappers-devel" >> Dockerfile'
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
      sh 'fpm -f -t rpm -s dir -n tacacs --version 1.0 --description "DD TacPlus" --depends pam-devel --depends tcp_wrappers -C $WORKSPACE/fpm-build .'
      sh 'rm -rf $WORKSPACE/fpm-build'
    }
  }
  stage('test') {
    docker.image('centos:7').inside('-u 0') {
      sh 'yum -y install $WORKSPACE/tacacs*.rpm'
    }
  }
}
