## mixin? ๐ค

ansible kubernetes ๋ชจ๋(`k8s`) ์์ค ์ฝ๋ ๋ถ์ ์ค, `K8sAnsibleMixin` ๋ผ๋ ์ด๋ฆ์ ํด๋์ค๊ฐ ์กด์ฌ. Mixin์ด๋ผ๋ ์ฉ์ด๋ฅผ vue์์๋ ๋ณธ ์ ์ด ์์ผ๋ ์ ํํ ์์ง ๋ชปํ์ฌ ๊ถ๊ธ์ฆ์ ๊ฒ์

#### mixin ์ํค ์ค๋ช ๐

> ๋ฏน์ค์ธ์ ์ฝ๋์ ์ฌ์ฌ์ฉ์ฑ์ ๋์ด๊ณ , ๋ค์ค์์์ผ๋ก ์ธํด ๋ฐ์ํ  ์ ์๋ ์์์ ๋ชจํธ์ฑ๋ฌธ์ (๋ค์ด์๋ชฌ๋ ๋ฌธ์ )๋ฅผ ์ ๊ฑฐํ๊ฑฐ๋ ์ธ์ด์์ ๋ค์ค์์์ ๋ํ ์ง์๋ถ์กฑ์ ํด๊ฒฐํ๊ธฐ ์ํด ์ฌ์ฉ๋  ์ ์์ต๋๋ค.

## ํ์ด์ฌ์์์ Mixin ๐

#### ๋ฏน์ค์ธ ํน์ง

- ์ถ๊ฐ์ ์ธ ๋ฉ์๋๋ง ์ ์ํ๋ _์์ ํด๋์ค_
- ํด๋์ค ๋ด๋ถ์ ์์ฑ(property)๋ฅผ ์ ์ํ์ง ์์
- `__init__` ์์ฑ์๋ฅผ ํธ์ถํ๋๋ก ์๊ตฌํ์ง ์์

#### When to use mixin

- ๋ค๋ฅธํด๋์ค์์ ์ฌ์ฉํ๊ณ ์ ํ๋ ํด๋์ค์ ํน์  ๊ธฐ๋ฅ๋ง ์ฌ์ฉํ๊ณ  ์ถ์๋
- ํด๋์ค๊ฐ ์ฌ์ฉํ  ๊ธฐ๋ฅ์ ๋ํ ์ ํ์ ์ ๊ณตํด์ฃผ๊ณ  ์ถ์ ๋, mixin ๊ณ ๋ ค

#### ๋ฏน์ค์ธ ์ฃผ์์ฌํญ

- ํ์ด์ฌ์ ๋์ผํ ๋ฉ์๋๊ฐ ์ค์ฒฉ๋ ๊ฒฝ์ฐ ๋ฎ์ด๋ฒ๋ฆผ(override)

```
class Mixin1(object):
    def test(self):
        print "Mixin1"

class Mixin2(object):
    def test(self):
        print "Mixin2"

class MyClass(BaseClass, Mixin1, Mixin2):
    pass



์ถ์ฒ: https://hamait.tistory.com/859 [HAMA ๋ธ๋ก๊ทธ]
```

- ํ์ด์ฌ์ ์ค๋ฅธ์ชฝ๋ถํฐ ์ผ์ชฝ์ผ๋ก ๊ณ์น๋จ
- ์ฆ, Mixin2๊ฐ ๊ฐ์ฅ ์์ ํด๋์ค
- ์ด๋ ๊ฐ์ฅ ํ์ ํด๋์ค(Mixin1)์ ๋ฉ์๋๊ฐ ์ ์ฉ๋จ

```
>>> obj = MyClass()
>>> obj.test()
Mixin1
```

## ansible k8s ์ฝ๋ mixin ํ์ฉ ์

#### kubernetes/core/plugins/module_utils/common.py

```
def main():
    module = AnsibleModule(argument_spec=argspec(), supports_check_mode=True)
    from ansible_collections.kubernetes.core.plugins.module_utils.common import (
        K8sAnsibleMixin, get_api_client)

    k8s_ansible_mixin = K8sAnsibleMixin(module)
    k8s_ansible_mixin.client = get_api_client(module=module)
    k8s_ansible_mixin.fail_json = module.fail_json
    k8s_ansible_mixin.fail = module.fail_json
    k8s_ansible_mixin.exit_json = module.exit_json
    k8s_ansible_mixin.warn = module.warn
    execute_module(module, k8s_ansible_mixin)
```

์ฝ๋๋ฅผ ๋ณด๋ฉด, `K8sAnsibleMixin`๋ ์ฟ ๋ฒ๋คํฐ์ค resource๋ฅผ ์กฐํ(`find_resource()`)ํ๊ฑฐ๋,  
์ฟ ๋ฒ๋คํฐ์ค resource๊ฐ ํน์  ์ํ("Running")๊ฐ ๋ ๋ ๊น์ง ๋๊ธฐํ๋ ๋ฉ์๋(`wait()`)๊ฐ ์ ์๋์ด ์์ผ๋ฉฐ  
์ด๋ ๋ค๋ฅธ ๋ฉ์๋๋ฅผ ์ด์ฉํ์ฌ ๊ตฌํ๋จ

```
class K8sAnsibleMixin(object):
...

    def find_resource(self, kind, api_version, fail=False):
        for attribute in ['kind', 'name', 'singular_name']:
            try:
                <!-- self.client ์ธ๋ถ์์ ๋ฐ์์จ ๊ฐ์ฒด -->
                return self.client.resources.get(**{'api_version': api_version, attribute: kind})
            except (ResourceNotFoundError, ResourceNotUniqueError):
                pass
        try:
            return self.client.resources.get(api_version=api_version, short_names=[kind])
        except (ResourceNotFoundError, ResourceNotUniqueError):
            if fail:
                self.fail(msg='Failed to find exact match for {0}.{1} by [kind, name, singularName, shortNames]'.format(api_version, kind))
...
```
