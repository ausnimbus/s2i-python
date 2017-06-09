/**
* NOTE: THIS JENKINSFILE IS GENERATED VIA "./hack/run update"
*
* DO NOT EDIT IT DIRECTLY.
*/
node {
        def variants = "default".split(',');
        for (int v = 0; v < variants.length; v++) {

                def versions = "2.7,3.6".split(',');
                for (int i = 0; i < versions.length; i++) {

                  if (variants[v] == "default") {
                    variant = ""
                    tag = "${versions[i]}"
                  } else {
                    variant = variants[v]
                    tag = "${versions[i]}-${variant}"
                  }


                        try {
                                stage("Build (Python-${tag})") {
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
                                                                        "name" : "${tag}",
                                                                        "from" : [
                                                                                "kind" : "DockerImage",
                                                                                "name" : "python:${tag}",
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
                                        "name" : "s2i-python-${tag}",
                                        "labels" : [
                                                "builder" : "s2i-python"
                                        ]
                                ],
                                "spec" : [
                                        "output" : [
                                                "to" : [
                                                        "kind" : "ImageStreamTag",
                                                        "name" : "s2i-python:${tag}"
                                                ]
                                        ],
                                        "runPolicy" : "Serial",
                                        "resources" : [
                                            "limits" : [
                                                "memory" : "2.5Gi"
                                            ]
                                        ],
                                        "source" : [
                                                "git" : [
                                                        "uri" : "https://github.com/ausnimbus/s2i-python"
                                                ],
                                                "type" : "Git"
                                        ],
                                        "strategy" : [
                                                "dockerStrategy" : [
                                                        "dockerfilePath" : "versions/${versions[i]}/${variant}/Dockerfile",
                                                        "from" : [
                                                                "kind" : "ImageStreamTag",
                                                                "name" : "python:${tag}"
                                                        ]
                                                ],
                                                "type" : "Docker"
                                        ]
                                ]
                        ])
        echo "Created s2i-python:${tag} objects"
        /**
        * TODO: Replace the sleep with import-image
        * openshift.importImage("python:${tag}")
        */
        sleep 60

        echo "==============================="
        echo "Starting build s2i-python-${tag}"
        echo "==============================="
        def builds = openshift.startBuild("s2i-python-${tag}");

        timeout(60) {
                builds.untilEach(1) {
                        return it.object().status.phase == "Complete"
                }
        }
        echo "Finished build ${builds.names()}"
        builds.logs()
}

                                }
                                stage("Test (Python-${tag})") {
                                        openshift.withCluster() {
        echo "==============================="
        echo "Starting test application"
        echo "==============================="

        def testApp = openshift.newApp("https://github.com/ausnimbus/python-ex", "--image-stream=s2i-python:${tag}", "-l app=python-ex");
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
                                stage("Stage (Python-${tag})") {
                                        openshift.withCluster() {
        echo "==============================="
        echo "Tag new image into staging"
        echo "==============================="

        openshift.tag("ausnimbus-ci/s2i-python:${tag}", "ausnimbus/s2i-python:${tag}")
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
}
