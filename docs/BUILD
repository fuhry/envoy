load(
    "//bazel:envoy_build_system.bzl",
    "envoy_package",
)

licenses(["notice"])  # Apache 2

exports_files([
    "protodoc_manifest.yaml",
    "v2_mapping.json",
    "empty_extensions.json",
    "redirects.txt",
])

envoy_package()

filegroup(
    name = "configs",
    srcs = glob(
        [
            "root/**/*.yaml",
        ],
        exclude = [
            "root/**/envoy-dynamic-cds-demo.yaml",
            "root/**/envoy-dynamic-lds-demo.yaml",
            "root/**/envoy-dynamic-filesystem-demo.yaml",
            # TODO(phlax/windows-dev): figure out how to get this working on windows
            #      "Error: unable to read file: /etc/ssl/certs/ca-certificates.crt"
            "root/configuration/http/http_filters/_include/dns-cache-circuit-breaker.yaml",
            "root/intro/arch_overview/security/_include/ssl.yaml",
        ],
    ) + select({
        "//bazel:windows_x86_64": [],
        "//conditions:default": [
            "root/configuration/http/http_filters/_include/dns-cache-circuit-breaker.yaml",
            "root/intro/arch_overview/security/_include/ssl.yaml",
        ],
    }),
)

filegroup(
    name = "proto_examples",
    srcs = glob(
        [
            "root/**/*.pb",
        ],
    ),
)

filegroup(
    name = "base_rst_files",
    srcs = glob(
        [
            "conf.py",
            "_ext/*",
        ],
    ),
)

filegroup(
    name = "root_rst_files",
    srcs = glob(["root/**/*"]),
)

genrule(
    name = "v2_redirects",
    outs = ["v2_redirects.txt"],
    cmd = "jq -r 'with_entries(.key |= sub(\"^envoy/\";\"api-v3/\")) | with_entries(.value |= sub(\"^envoy/\";\"api-v2/\")) | to_entries[] | \"\\(.value)\t\t\\(.key)\"' docs/v2_mapping.json > $@",
    tools = ["//docs:v2_mapping.json"],
)

genrule(
    name = "redirects",
    outs = ["envoy-redirects.txt"],
    cmd = "cat $(location //docs:redirects.txt) > $@ && cat $(location :v2_redirects) >> $@",
    tools = [
        ":redirects.txt",
        ":v2_redirects",
    ],
)

genrule(
    name = "google_vrp_config",
    outs = ["google_vrp_config.tar"],
    cmd = "tar cfh $@ --transform='s/configs/configuration\\/best_practices/' $(location //configs:google-vrp/envoy-edge.yaml)",
    tools = [
        "//configs:google-vrp/envoy-edge.yaml",
    ],
)

genrule(
    name = "base_rst",
    outs = ["base_rst.tar"],
    cmd = "tar cfh $@ --transform='s/docs\\///' $(locations base_rst_files) && tar -rh --file=$@ --transform='s/docs\\/root\\///' $(locations root_rst_files) && tar -r --file=$@ -C $$(dirname $(location redirects)) $$(basename $(location redirects))",
    tools = [
        ":base_rst_files",
        ":redirects",
        ":root_rst_files",
    ],
)

genrule(
    name = "examples_rst",
    outs = ["examples_rst.tar"],
    cmd = "tar cfh $@ --transform='s/examples/start\\/sandboxes\\/_include/' $(locations //examples:files)",
    tools = [
        "//examples:files",
    ],
)
