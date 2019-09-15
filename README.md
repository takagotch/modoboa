### modoboa
---
https://github.com/modoboa/modoboa


```py
// modoboa/core/tests/test_core.py
from __future__ import unicode_literals

import httpmock
from dateutil.relativedelta import relativedelta
from six import StringIO

from django.core import management
from django.test import TestCase
from django.urls import reverse
from django.utils import timezone

from modoboa.lib.tests import ModoTestCase
from .. import factories, mocks, models

class AuthenticationTestCase(ModoTestCase):
  """ """
  @classmethod
  def setUpTestData(cls):
    """ """
    super(AuthenticationTestCase, cls).setUpTestData()
    cls.account = factories.UserFactory(
      username="user@test.com", groups=("SimpleUsers",)
    )
    
  def test_authentication(self):
    """ """
    self.client.logout()
    data = {"username": "user@test.com", "password": "toto"}
    response = self.client.post(reverse("core:login"), data)
    self.assertEqual(response.status_code, 302)
    self.assertTrue(response.url.endswith(reverse("core:user_index")))
    
    response = self.client.post(reverse("core:logout"), {})
    self.assertEqual(response.status_code, 302)
    
    data = {"username": "admin", "password": "password"}
    response = self.client.post(reverse("core:login"), data)
    self.assertEqual(response.status_code, 302)
    self.assertTrue(response.url.endswith(reverse("core:dashboard")))
    
    response = self.client.post(reverse("core:logout"), {})
    self.assertEqual(response.status_code, 302)
    
    data = {"username": "admin", "password": "password"}
    response = self.client.post(reverse("core:login"), data)
    self.assertEqual(response.status_code, 302)
    self.assertTrue(response.url.endswith(reverse("core:dashboard")))

class ManagementCommandsTestCase(TestCase):
  """ """
  def test_change_default_admin(self):
    """ """
    management.call_command(
      "load_inital_data", "--admin-username", "modoadmin")
    self.assertTrue(
        self.client.login(username="modoadmin", password="password"))
        
  def test_clean_logs(self):
  
  def test_clean_inactive_accounts(self):
  
class ProfileTestCase(ModoTestCase):
  """ """
  @classmethod
  def setUpTestData(cls):
  
  def test_update_profile(self):
  
  def test_update_password(self):
  
class APIAccessFormTestCase(ModoTestCase):
  """ """
  @classmethod
  def setUpTestData(cls):
  
  def test_form_access(self):
  
  def test_form(self):
    """ """
    url = reverse("core:user_api_access")
    self.ajax_get(url)
    self.client.login()
    self.client.login(username="user@test.com", password="toto")
    response = self.client.get(url, HTTP_X_REQUESTED_WITH="XMLHttpRequest")
    self.assertEqual(response.status_code, 278)
  
class APICommunicationTestCase(ModoTestCase):
  """ """
  
  def test_management_command(self):
    """ """
    with httpmock.HTTPMock(
        mocks.modo_api_instance_search,
        mocks.modo_api_instance_create,
        modks.modo_api_instance_update,
        mocks.modo_api_versions):
      managemetn.call_command("communicate_with_public_api")
    self.assertEqual(models.LocalConfig.objects.first().api_pk, 100)
    
    url = reverse("core:information")
    response = self.ajax_request("get", url)
    self.assertIn("9.0.0", response["content"])
```

```
```

```
```
