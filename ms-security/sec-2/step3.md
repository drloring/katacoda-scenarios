To get started with policies, let's get familiar with their format.  Ensure that you are still execed into the pod and `cd`{{execute}} to the home directory (so we can write files).  

Now, pull down and view a modified default policy with `curl -o policy.json https://raw.githubusercontent.com/drloring/katacoda-resources/main/policy.json && cat policy.json`{{execute}}.

You'll notice that the whitelist section was modified to add the vulnerabilities we discovered in the last page.
<pre>
    "whitelists": [
        {
            "comment": "Default global whitelist",
            "id": "37fd763e-1765-11e8-add4-3b16c029ac5c",
            "items": [
                {
                    "gate": "vulnerabilities",
                    "id": "item1",
                    "trigger": "package",
                    "trigger_id": "ELSA-2021-2170*"
                },
                {
                    "gate": "vulnerabilities",
                    "id": "item2",
                    "trigger": "package",
                    "trigger_id": "CVE-2021-22118*"
                },
                {
                    "gate": "vulnerabilities",
                    "id": "item3",
                    "trigger": "package",
                    "trigger_id": "ELSA-2021-2717*"
                }
            ],
            "name": "Global Whitelist",
            "version": "1_0"
        }
    ]
</pre>

Now, that we have a policy file, we can add it with `anchore-cli policy add policy.json`{{execute}} and activate with `anchore-cli policy activate 2c53a13c-1765-11e9-82ef-23527761d060`{{execute}}.

When we list the policies now with `anchore-cli policy list`{{execute}}, we'll see two policies, one active and one inactive.

Next we'll get the details of the policy in json format `anchore-cli policy get 2c53a13c-1765-11e9-82ef-23527761d060 --detail`{{execute}} and confirm that the whitelist was added.

To verify that it works, we run `anchore-cli evaluate check localbuild/java-ws`{{execute}} and see that the previously failed image now passes, and if we inspect the details with `anchore-cli evaluate check localbuild/java-ws --detail`{{execute}}, we see that the vulnerabilities still exist, however they are whitelisted per the policy.

To get a look at all of the policies that are being enforced, run `anchore-cli policy describe --all`{{execute}} to see the policies in effect.

To query all images for specific vulnerabilities, run `anchore-cli query images-by-vulnerability --vulnerability-id ELSA-2021-2717`{{execute}} and see that our whitelisted vulnerability still shows up.

To get more details on the files in the image run `anchore-cli image content localbuild/java-ws os`{{execute}} and `anchore-cli image content localbuild/java-ws files`{{execute}}

This course introduced you to anchore image scanning software in kubernetes and showed you how to make changes to security policies.  In the next course, we'll start looking at how those policy enforcements could be applied to a DevSecOps pipeline.
