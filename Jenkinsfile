/**
* NOTE: THIS JENKINSFILE IS GENERATED VIA "./hack/run update"
*
* DO NOT EDIT IT DIRECTLY.
*/
node {
        def versions = "2.7,3.6".split(',');
        for (int i = 0; i < versions.length; i++) {
                try {
                        stage("Build (Python ${versions[i]})") {
                                openshift.withCluster() {
        openshift.apply([
                                "apiVersion" : "v1",
                                "items" : [
                                        [
                                                "apiVersion" : "v1",
                                                "kind" : "ImageStream",
                                                "metadata" : [
                                                        "name" : "python",
                                                        "labels" : [
                                                                "builder" : "s2i-python"
                                                        ]
                                                ],
                                                "spec" : [
                                                        "tags" : [
                                                                [
                                                                        "name" : "${versions[i]}-alpine",
                                                                        "from" : [
                                                                                "kind" : "DockerImage",
                                                                                "name" : "python:${versions[i]}-alpine",
                                                                        ],
                                                                        "referencePolicy" : [
                                                                                "type" : "Source"
                                                                        ]
                                                                ]
                                                        ]
                                                ]
                                        ],
                                        [
                                                "apiVersion" : "v1",
                                                "kind" : "ImageStream",
                                                "metadata" : [
                                                        "name" : "s2i-python",
                                                        "labels" : [
                                                                "builder" : "s2i-python"
                                                        ]
                                                ]
                                        ]
                                ],
                                "kind" : "List"
                        ])
        openshift.apply([
                                "apiVersion" : "v1",
                                "kind" : "BuildConfig",
                                "metadata" : [
                                        "name" : "s2i-python-${versions[i]}",
                                        "labels" : [
                                                "builder" : "s2i-python"
                                        ]
                                ],
                                "spec" : [
                                        "output" : [
                                                "to" : [
                                                        "kind" : "ImageStreamTag",
                                                        "name" : "s2i-python:${versions[i]}"
                                                ]
                                        ],
                                        "runPolicy" : "Serial",
                                        "source" : [
                                                "git" : [
                                                        "uri" : "https://github.com/ausnimbus/s2i-python"
                                                ],
                                                "type" : "Git"
                                        ],
                                        "strategy" : [
                                                "dockerStrategy" : [
                                                        "dockerfilePath" : "versions/${versions[i]}/Dockerfile",
                                                        "from" : [
                                                                "kind" : "ImageStreamTag",
                                                                "name" : "python:${versions[i]}-alpine"
                                                        ]
                                                ],
                                                "type" : "Docker"
                                        ]
                                ]
                        ])
        echo "Created s2i-python:${versions[i]} objects"
        /**
        * TODO: Replace the sleep with import-image
        * openshift.importImage("python:${versions[i]}-alpine")
        */
        sleep 60

        echo "==============================="
        echo "Starting build s2i-python-${versions[i]}"
        echo "==============================="
        def builds = openshift.startBuild("s2i-python-${versions[i]}");

        timeout(10) {
                builds.untilEach(1) {
                        return it.object().status.phase == "Complete"
                }
        }
        echo "Finished build ${builds.names()}"
}

                        }
                        stage("Test (Python ${versions[i]})") {
                                openshift.withCluster() {
        echo "==============================="
        echo "Starting test application"
        echo "==============================="

        def testApp = openshift.newApp("https://github.com/ausnimbus/python-ex", "--image-stream=s2i-python:${versions[i]}", "-l app=python-ex");
        echo "new-app created ${testApp.count()} objects named: ${testApp.names()}"
        testApp.describe()

        def testAppBC = testApp.narrow("bc");
        def testAppBuilds = testAppBC.related("builds");
        echo "Waiting for ${testAppBuilds.names()} to finish"
        timeout(10) {
                testAppBuilds.untilEach(1) {
                        return it.object().status.phase == "Complete"
                }
        }
        echo "Finished ${testAppBuilds.names()}"

        def testAppDC = testApp.narrow("dc");
        echo "Waiting for ${testAppDC.names()} to start"
        timeout(10) {
                testAppDC.untilEach(1) {
                        return it.object().status.availableReplicas >= 1
                }
        }
        echo "${testAppDC.names()} is ready"

        def testAppService = testApp.narrow("svc");
        def testAppHost = testAppService.object().spec.clusterIP;
        def testAppPort = testAppService.object().spec.ports[0].port;

        sleep 60
        echo "Testing endpoint ${testAppHost}:${testAppPort}"
        sh "curl -o /dev/null $testAppHost:$testAppPort"
}

                        }
                } finally {
                        openshift.withCluster() {
                                echo "Deleting test resources python-ex"
                                openshift.selector("dc", [app: "python-ex"]).delete()
                                openshift.selector("bc", [app: "python-ex"]).delete()
                                openshift.selector("svc", [app: "python-ex"]).delete()
                                openshift.selector("is", [app: "python-ex"]).delete()
                                openshift.selector("pods", [app: "python-ex"]).delete()
                                openshift.selector("routes", [app: "python-ex"]).delete()
                        }
                }

        }
}
