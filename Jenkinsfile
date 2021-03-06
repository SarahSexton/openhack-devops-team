podTemplate(
    label: 'hashicorp',
    containers: [
        containerTemplate(name: 'packer', image: 'hashicorp/packer:1.1.3', ttyEnabled: true, command: 'cat'),
        containerTemplate(
          name: 'terraform',
          image: 'hashicorp/terraform:0.11.1',
          ttyEnabled: true,
          command: 'cat'
        )
    ],
    envVars: [
        envVar(key: 'IMAGE_RESOURCE_GROUP_NAME', value: 'apache'),
        envVar(key: 'BUILD_RESOURCE_GROUP_NAME', value: 'apache'),
        envVar(key: 'IMAGE_NAME_PREFIX', value: 'apachecustom'),
        secretEnvVar(key: 'ARM_CLIENT_ID', secretName: 'azuresecrets', secretKey: 'ARM_CLIENT_ID'),
        secretEnvVar(key: 'ARM_CLIENT_SECRET', secretName: 'azuresecrets', secretKey: 'ARM_CLIENT_SECRET'),
        secretEnvVar(key: 'ARM_STORAGE_ACCOUNT', secretName: 'azuresecrets', secretKey: 'ARM_STORAGE_ACCOUNT'),
        secretEnvVar(key: 'ARM_SUBSCRIPTION_ID', secretName: 'azuresecrets', secretKey: 'ARM_SUBSCRIPTION_ID'),
        secretEnvVar(key: 'ARM_TENANT_ID', secretName: 'azuresecrets', secretKey: 'ARM_TENANT_ID'),
        secretEnvVar(key: 'STORAGE_URL', secretName: 'azuresecrets', secretKey: 'STORAGE_URL'),
        secretEnvVar(key: 'STORAGE_USER', secretName: 'azuresecrets', secretKey: 'STORAGE_USER'),
        secretEnvVar(key: 'STORAGE_PASSWORD', secretName: 'azuresecrets', secretKey: 'STORAGE_PASSWORD')
    ],
    volumes: [
        secretVolume(secretName: 'tfbackend', mountPath: '/etc/tfvars'),
        secretVolume(secretName: 'sshkey', mountPath: '/home/jenkins/.ssh')
        ]
    ) {

    node('hashicorp') {
        stage('Packer CI') {
            git 'https://github.com/dtzar/JenkinsPipeTest.git'
            container('packer') {
                stage('Build Packer Image') {
                    echo "Building $IMAGE_NAME_PREFIX${env.BUILD_ID}"
                    sh """
                    export IMAGE_NAME=$IMAGE_NAME_PREFIX${env.BUILD_ID}
                    packer build ./httpd/httpd.json
                    """
                }
            }
        }

        stage('Terraform CD') {
            git url: 'https://github.com/dtzar/JenkinsPipeTest.git'
            container('terraform') {
                stage('Deploy Packer Image with Terraform') {
                    sh """
                    cd httpd
                    terraform init -backend-config=/etc/tfvars/beconf.tfvars
                    terraform apply -var 'location=eastus' -var 'packer_image=$IMAGE_NAME_PREFIX${env.BUILD_ID}' -var 'storage_url=$STORAGE_URL' -var 'storage_user=$STORAGE_USER' -var 'storage_password=$STORAGE_PASSWORD' -auto-approve
                    """
                }
            }
        }

    }
}