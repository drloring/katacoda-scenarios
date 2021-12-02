To get started with policies, let's get familiar with their format.  If you're still execed into the pod, exit out of the anchore-cli pod exit{{execute}}, pull down and view a modified default policy with `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/policy.json && cat policy.json`{{execute}}.

You'll notice that the section regarding the HEALTHCHECK was modified
<pre>
                    "action": "STOP",
                    "gate": "dockerfile",
                    "id": "312d9e41-1c05-4e2f-ad89-b7d34b0855bb",
                    "params": [
                        {
                            "name": "instruction",
                            "value": "HEALTHCHECK"
                        },
                        {
                            "name": "check",
                            "value": "not_exists"
                        }
                    ],
                    "trigger": "instruction"
                },
</pre>

Next, copy the entire json file text to your clipboard in the browser and exec back into the anchore-cli `kubectl exec -it anchore-cli -- bash`{{execute}}.  From there, `cd && vi policy.json`{{execute}} and now put vi into insert mode `i`{{execute}} and paste the contents of the file, hit `Esc` and `:wq`{{execute}} to save the file.

Now, that we have a policy file, we can add it with `anchore-cli policy add policy.json`{{execute}} and activate with `anchore-cli policy activate 2c53a13c-1765-11e9-82ef-23527761d060`{{execute}}.

When we list the policies now with `anchore-cli policy list`{{execute}}, we'll see two policies, one active and one inactive.

Next we'll get the details of the policy in json format `anchore-cli policy get 2c53a13c-1765-11e9-82ef-23527761d060 --detail`{{execute}}.  If you recall, previously, the redis image scan issued a warning for not having a `HEALTHCHECK` in the Dockerfile.  Well, we take our health seriously, so we changed that to a `STOP` with the imported policy bundle.

To verify, we run `anchore-cli evaluate check redis`{{execute}} and see that the previously passed image now fails based on the policy change we made.

This course introduced you to anchore image scanning software in kubernetes and showed you how to make changes to security policies.  In the next course, we'll start looking at how those policy enforcements could be applied to a DevSecOps pipeline.
