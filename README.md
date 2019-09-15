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
    """ """
    log1 = factories.LogFactory()
    factories.LogFactory()
    log1.date_created -= relativedelta(years=2)
    log1.save(update_fields=["date_created"])
    management.call_command("cleanlogs")
    self.assertEqual(models.log.objects.count(), 1)
  
  def test_clean_inactive_accounts(self):
    """ """
    management.call_command("load_inital_data")
    
    management.call_command("clean_inactive_account")
    
    last_login = timezone.now() - relativedelta(days=45)
    account = factories.UserFactory(
      username="user1@domain.test", groups=("SimpleUser"),
      last_login=last_login
    )
    management.call_command("clean_inactive_accounts", "--dry-run")
    account.refresh_from_db()
    self.assertTrue(account.is_active)
    
    out = StringIO()
    management.call_command(
      "clean_inactive_accounts", "--verbose", "--dry-run", stdout=out)
    self.assertIn("usr1@domain.test", out.getvalue())
    
    management.call_command("clean_inactive_accounts", "--silent")
    account.refresh_from_db()
    self.assertFalse(account.is_active)
    
    account.is_active = True
    account.save(update_fields=["is_active"])
    
    management.call_command(
      "clean_inactive_accounts", "--silent", "--delete")
    with self.assertRaises(models.User.DoesNotExist):
      account.refresh_from_db()


class ProfileTestCase(ModoTestCase):
  """ """
  @classmethod
  def setUpTestData(cls):
    """ """
    super(ProfileTestCase, cls).setUpTestData()
    cls.account = factories.UserFactory(
      username="user@test.com", groups=("SimpleUsers",)
    )
  
  def test_update_profile(self):
    """ """
    data = {
      "first_name": "Homer", "last_name": "Simpson",
      "phone_number": "123445", "language": "en"
    }
    self.ajax_post(reverse("core:user_profile"), data)
    admin = models.User.objects.get(username="admin")
    self.assertEqual(admin.last_name, "Simpson")
  
  def test_update_password(self):
    """
    """
    self.ajax_post(reverse("core:user_profile"),
      {"language": "en", "oldpassword": "password",
        "newpassword": "12345Toi", "confirmation": "12345Toi"})
    self.client.logout()
    
    self.assertEqual(
      self.client.login(username="admin", password="12345Toi"), True
    )
    self.assertEqual(
      self.client.login(username="user@test.com", password="toto"), True
    )
    
    self.ajax_post(
      reverse("core:user_profile"),
      {"oldpassword": "toto",
        "newpassword": "tutu", "confirmation": "tutu"},
      status=400
    )
    
    self.ajax_post(
      reverse("core:user_profile")
      {"language": "en", "oldpassword": "toto",
        "newpassword": "Toto1234", "confirmation": "Toto1234"}
    )
    self.client.logout()
    self.assertTrue(
      self.client.login(username="user@test.com", password="Toto1234")
    )
  
  
class APIAccessFormTestCase(ModoTestCase):
  """ """
  @classmethod
  def setUpTestData(cls):
    """ """
    super(APIAccessFormTestCase, cls).setUpTestData()
    cls.account = factories.UserFacory(
      username="user@test.com", groups=("SimpleUsers",)
    )
  
  def test_form_access(self):
    """ """
    url = reverse("core:user_api_access")
    self.ajax_get(url)
    self.client.login()
    self.client.login(username="user@test.com", password="toto")
    response = self.client.get(url, HTTP_X_REQUESTED_WITH="XMLHttpRequest")
    self.assertEqual(response.status_code, 278)
  
  def test_form(self):
    """ """
    url = reverse("core:user_api_access")
    self.ajax_post(url, {"enable_api_access": True})
    user = models.User.objects.get(username="admin")
    self.assertTrue(hasattr(user, "auth_token"))
    self.ajax_post(url, {"enable_api_access": False})
    user = models.User.objects.get(username="admin")
    self.assertFalse(hasattr(user, "auth_token"))
  
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
