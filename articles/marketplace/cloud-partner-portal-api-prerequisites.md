---
title: Wymagania wstępne dotyczące interfejsu API — Azure Marketplace
description: Wymagania wstępne dotyczące korzystania z portal Cloud Partner interfejsów API.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 09/23/2020
ms.openlocfilehash: f5fc77b65f6a83f4f7ca8ed8b8c9294b95307735
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107316"
---
# <a name="api-prerequisites"></a>Wymagania wstępne dotyczące interfejsu API

> [!NOTE]
> Interfejsy API portal Cloud Partner są zintegrowane z usługą i będą nadal działać w centrum partnerskim. Przejście wprowadza niewielkie zmiany. Przejrzyj zmiany wymienione w [dokumentacji interfejsu API Portal Cloud partner](cloud-partner-portal-api-overview.md) , aby upewnić się, że kod będzie kontynuował pracę po przejściu do Centrum partnerskiego. Używaj tylko interfejsów API CPP dla istniejących produktów, które zostały już zintegrowane przed przejściem do Centrum partnerskiego; nowe produkty powinny używać interfejsów API przekazywania Centrum partnerskiego.

Potrzebne są dwa wymagane zasoby programistyczne do używania portal Cloud Partner interfejsów API: nazwy głównej usługi i tokenu dostępu Azure Active Directory (Azure AD).

## <a name="create-service-principal-in-azure-active-directory-tenant"></a>Tworzenie nazwy głównej usługi w dzierżawie Azure Active Directory

Najpierw musisz utworzyć nazwę główną usługi w dzierżawie usługi Azure AD. Do tej dzierżawy zostanie przypisany własny zestaw uprawnień w portal Cloud Partner. Kod będzie wywoływał interfejsy API korzystające z tej dzierżawy zamiast poświadczeń osobistych. Aby uzyskać pełne wyjaśnienie tworzenia nazwy głównej usługi, zobacz [How to: Use the Portal, aby utworzyć aplikację usługi Azure AD i nazwę główną usługi, która może uzyskiwać dostęp do zasobów](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="add-service-principal-to-your-account"></a>Dodawanie nazwy głównej usługi do konta

Teraz, po utworzeniu jednostki usługi w dzierżawie, możesz ją dodać jako użytkownika do konta portalu Centrum partnerskiego. Podobnie jak użytkownik, nazwa główna usługi może być właścicielem lub współautorem portalu. Aby uzyskać szczegółowe informacje, zobacz **następne kroki** poniżej.

## <a name="next-steps"></a>Następne kroki

Zobacz [Zarządzanie aplikacjami usługi Azure AD](manage-aad-apps.md).
