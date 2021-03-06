---
title: 'Samouczek: Azure Active Directory integrację logowania jednokrotnego (SSO) z usługą PerimeterX | Microsoft Docs'
description: Dowiedz się, jak skonfigurować Logowanie jednokrotne między Azure Active Directory i PerimeterX.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/27/2020
ms.author: jeedes
ms.openlocfilehash: 29ffaaa1b1b6efbcd5523a76018c92645e13d187
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/29/2021
ms.locfileid: "96181802"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-perimeterx"></a>Samouczek: Azure Active Directory integracji logowania jednokrotnego (SSO) z usługą PerimeterX

W tym samouczku dowiesz się, jak zintegrować usługę PerimeterX z usługą Azure Active Directory (Azure AD). Po zintegrowaniu usługi PerimeterX z usługą Azure AD można:

* Kontrolka w usłudze Azure AD, która ma dostęp do PerimeterX.
* Zezwól użytkownikom na automatyczne logowanie się do usługi PerimeterX przy użyciu kont w usłudze Azure AD.
* Zarządzaj kontami w jednej centralnej lokalizacji — Azure Portal.

## <a name="prerequisites"></a>Wymagania wstępne

Aby rozpocząć, potrzebne są następujące elementy:

* Subskrypcja usługi Azure AD. Jeśli nie masz subskrypcji, możesz uzyskać [bezpłatne konto](https://azure.microsoft.com/free/).
* Subskrypcja z włączonym logowaniem jednokrotnym (SSO) PerimeterX.

## <a name="scenario-description"></a>Opis scenariusza

W tym samouczku skonfigurujesz i testujesz Logowanie jednokrotne usługi Azure AD w środowisku testowym.


* Usługa PerimeterX obsługuje **dostawcy tożsamości** zainicjowane przez logowanie jednokrotne

## <a name="adding-perimeterx-from-the-gallery"></a>Dodawanie PerimeterX z galerii

Aby skonfigurować integrację programu PerimeterX z usługą Azure AD, musisz dodać PerimeterX z galerii do listy zarządzanych aplikacji SaaS.

1. Zaloguj się do Azure Portal przy użyciu konta służbowego lub konto Microsoft prywatnego.
1. W okienku nawigacji po lewej stronie wybierz usługę **Azure Active Directory** .
1. Przejdź do **aplikacji przedsiębiorstwa** , a następnie wybierz pozycję **wszystkie aplikacje**.
1. Aby dodać nową aplikację, wybierz pozycję **Nowa aplikacja**.
1. W sekcji **Dodaj z galerii** wpisz **PerimeterX** w polu wyszukiwania.
1. Wybierz pozycję **PerimeterX** from panel wyników, a następnie Dodaj aplikację. Poczekaj kilka sekund, gdy aplikacja zostanie dodana do dzierżawy.


## <a name="configure-and-test-azure-ad-sso-for-perimeterx"></a>Skonfiguruj i przetestuj Logowanie jednokrotne usługi Azure AD dla PerimeterX

Skonfiguruj i przetestuj Logowanie jednokrotne usługi Azure AD za pomocą PerimeterX przy użyciu użytkownika testowego o nazwie **B. Simon**. Aby logowanie jednokrotne działało, należy ustanowić relację linku między użytkownikiem usługi Azure AD i powiązanym użytkownikiem w PerimeterX.

Aby skonfigurować i przetestować Logowanie jednokrotne usługi Azure AD za pomocą PerimeterX, wykonaj następujące czynności:

1. **[Skonfiguruj Logowanie jednokrotne usługi Azure AD](#configure-azure-ad-sso)** , aby umożliwić użytkownikom korzystanie z tej funkcji.
    1. **[Utwórz użytkownika testowego usługi Azure AD](#create-an-azure-ad-test-user)** — aby przetestować Logowanie jednokrotne w usłudze Azure AD za pomocą usługi B. Simon.
    1. **[Przypisz użytkownika testowego usługi Azure AD](#assign-the-azure-ad-test-user)** — aby umożliwić usłudze B. Simon korzystanie z logowania jednokrotnego w usłudze Azure AD.
1. **[Skonfiguruj Logowanie jednokrotne](#configure-perimeterx-sso)** w usłudze PerimeterX, aby skonfigurować ustawienia logowania jednokrotnego na stronie aplikacji.
    1. **[Utwórz użytkownika testowego PerimeterX](#create-perimeterx-test-user)** , aby dysponować odpowiednikiem B. Simon w PerimeterX, która jest połączona z reprezentacją użytkownika w usłudze Azure AD.
1. **[Przetestuj Logowanie jednokrotne](#test-sso)** — aby sprawdzić, czy konfiguracja działa.

## <a name="configure-azure-ad-sso"></a>Konfigurowanie rejestracji jednokrotnej w usłudze Azure AD

Wykonaj następujące kroki, aby włączyć logowanie jednokrotne usługi Azure AD w Azure Portal.

1. W Azure Portal na stronie integracja aplikacji **PerimeterX** Znajdź sekcję **Zarządzanie** i wybierz pozycję **Logowanie jednokrotne**.
1. Na stronie **Wybierz metodę logowania jednokrotnego** wybierz pozycję **SAML**.
1. Na stronie **Konfigurowanie logowania jednokrotnego przy użyciu języka SAML** kliknij ikonę Edytuj/pióro, aby określić **podstawową konfigurację języka SAML** , aby edytować ustawienia.

   ![Edycja podstawowej konfiguracji protokołu SAML](common/edit-urls.png)

1. W sekcji **Podstawowa konfiguracja języka SAML** aplikacja została wstępnie skonfigurowana i wymagane adresy URL są już wstępnie wypełnione na platformie Azure. Użytkownik musi zapisać konfigurację, klikając przycisk **Zapisz** .


1. Aplikacja PerimeterX oczekuje potwierdzeń SAML w określonym formacie, co wymaga dodania niestandardowych mapowań atrybutów do konfiguracji atrybutów tokenu SAML. Poniższy zrzut ekranu przedstawia listę atrybutów domyślnych.

    ![image (obraz)](common/default-attributes.png)

1. Oprócz powyższych, aplikacja PerimeterX oczekuje kilku atrybutów do przekazania z powrotem w odpowiedzi SAML, które przedstawiono poniżej. Te atrybuty są również wstępnie wypełnione, ale można je sprawdzić zgodnie z wymaganiami.
    
    | Nazwa | Atrybut źródłowy|
    | ------------ | --------- |
    | firstName | user.givenname |
    | lastName  | user.surname |

1. Na stronie **Konfigurowanie logowania jednokrotnego przy użyciu języka SAML** w sekcji **certyfikat podpisywania SAML** Znajdź **certyfikat (base64)** i wybierz pozycję **Pobierz** , aby pobrać certyfikat i zapisać go na komputerze.

    ![Link do pobierania certyfikatu](common/certificatebase64.png)

1. W sekcji **Konfigurowanie PerimeterX** skopiuj odpowiednie adresy URL na podstawie wymagania.

    ![Kopiowanie adresów URL konfiguracji](common/copy-configuration-urls.png)
### <a name="create-an-azure-ad-test-user"></a>Tworzenie użytkownika testowego usługi Azure AD

W tej sekcji utworzysz użytkownika testowego w Azure Portal o nazwie B. Simon.

1. W lewym okienku w Azure Portal wybierz pozycję **Azure Active Directory**, wybierz pozycję **Użytkownicy**, a następnie wybierz pozycję **Wszyscy użytkownicy**.
1. Wybierz pozycję **nowy użytkownik** w górnej części ekranu.
1. We właściwościach **użytkownika** wykonaj następujące kroki:
   1. W polu **Nazwa** wprowadź wartość `B.Simon`.  
   1. W polu **Nazwa użytkownika** wprowadź wartość username@companydomain.extension . Na przykład `B.Simon@contoso.com`.
   1. Zaznacz pole wyboru **Pokaż hasło** i zanotuj wartość wyświetlaną w polu **Hasło**.
   1. Kliknij pozycję **Utwórz**.

### <a name="assign-the-azure-ad-test-user"></a>Przypisywanie użytkownika testowego usługi Azure AD

W tej sekcji włączysz usługę B. Simon, aby korzystać z logowania jednokrotnego na platformie Azure przez przyznanie dostępu do usługi PerimeterX.

1. W Azure Portal wybierz pozycję **aplikacje dla przedsiębiorstw**, a następnie wybierz pozycję **wszystkie aplikacje**.
1. Na liście Aplikacje wybierz pozycję **PerimeterX**.
1. Na stronie Przegląd aplikacji Znajdź sekcję **Zarządzanie** i wybierz pozycję **Użytkownicy i grupy**.
1. Wybierz pozycję **Dodaj użytkownika**, a następnie w oknie dialogowym **Dodawanie przypisania** wybierz pozycję **Użytkownicy i grupy** .
1. W oknie dialogowym **Użytkownicy i grupy** wybierz pozycję **B. Simon** z listy Użytkownicy, a następnie kliknij przycisk **Wybierz** w dolnej części ekranu.
1. Jeśli oczekujesz, że rola ma być przypisana do użytkowników, możesz wybrać ją z listy rozwijanej **Wybierz rolę** . Jeśli nie skonfigurowano roli dla tej aplikacji, zostanie wyświetlona wybrana rola "domyślny dostęp".
1. W oknie dialogowym **Dodawanie przypisania** kliknij przycisk **Przypisz** .

## <a name="configure-perimeterx-sso"></a>Konfigurowanie logowania jednokrotnego PerimeterX

1. Zaloguj się do konsoli PerimeterX z uprawnieniami administratora.

1. Przejdź do **konta administratora >** 

    ![PerimeterX Logowanie jednokrotne](./media/perimeterx-tutorial/accounts.png)

1. Kliknij przycisk **Edytuj**

4.  W oknie dialogowym Edytowanie konta wykonaj następujące czynności.

    ![Konto edycji logowania jednokrotnego w usłudze PerimeterX](./media/perimeterx-tutorial/saml-sso.png)

    a.  Zaznacz **opcję Włącz pojedyncze Sign-On (SSO)**

    b.  Wybierz pozycję **Azure SAML**.

    c.  W polu tekstowym **punkt końcowy SAML** wklej wartość **adresu URL logowania** , którą skopiowano Azure Portal.

    d. W polu tekstowym **wystawca** wklej wartość identyfikatora usługi Azure AD skopiowaną z Azure Portal.

    e. Otwórz pobrany **certyfikat (base64)** z Azure Portal do Notatnika i wklej zawartość do pola tekstowego **certyfikatu X. 509** .

    f. Kliknij pozycję **Zapisz zmiany**

### <a name="create-perimeterx-test-user"></a>Utwórz użytkownika testowego PerimeterX

Instrukcje dotyczące tworzenia użytkownika testowego PerimeterX można znaleźć w [podręczniku zarządzanie użytkownikami PerimeterX](https://docs.perimeterx.com/pxconsole/docs/managing-users) .

## <a name="test-sso"></a>Testuj Logowanie jednokrotne 

W tej sekcji przetestujesz konfigurację logowania jednokrotnego usługi Azure AD przy użyciu następujących opcji.

1. Kliknij pozycję Testuj tę aplikację w Azure Portal i należy automatycznie zalogować się do PerimeterX, dla którego skonfigurowano Logowanie jednokrotne

1. Możesz użyć panelu programu Microsoft Access. Po kliknięciu kafelka PerimeterX w panelu dostępu należy automatycznie zalogować się do PerimeterX, dla którego skonfigurowano Logowanie jednokrotne. Aby uzyskać więcej informacji na temat panelu dostępu, zobacz [wprowadzenie do panelu dostępu](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Następne kroki

Po skonfigurowaniu PerimeterX można wymusić kontrolę sesji, co chroni eksfiltracji i niefiltrowanie danych poufnych organizacji w czasie rzeczywistym. Kontrolka sesji rozciąga się od dostępu warunkowego. [Dowiedz się, jak wymuszać kontrolę sesji za pomocą Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).