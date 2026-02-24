==========================================================================================================================
                                        Inspect What Helm Generated
==========================================================================================================================

1. Rendered YAML

    helm get manifest my-nginx

Purpose: See actual Kubernetes YAML Helm sent to API server.

2. Release Values

    helm get values my-nginx

Purpose: Shows user-provided values (overrides)

3. Full Values (Including Defaults)

    helm get values my-nginx --all