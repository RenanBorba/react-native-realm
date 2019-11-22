# proj-react-mobile-realm
# Projeto Portfólio - Aplicação RepOff Mobile React Native com o RealmDB (Offline Storage)
Aplicação Front-end Mobile desenvolvida em React Native voltada para a busca de respositórios no Github, permitindo assim adicionar repositórios ao aplicativo e consultá-los de maneira offline. 
<ul> 
  <li>Realm</li>
  <li>Axios</li>
  <li>Github API repos</li>
  <li>react-native-vector-icons</li>
  <li>react-native-linear-gradient</li>
  <li>Components</li>
  <li>styled-components</li>
  <li>FlatList</li>
  <li>useState</li>
  <li>useEffect</li>
</ul>
<br>

## src/pages/Main/index.js 

```js
import React, { useState, useEffect } from 'react';
import { Keyboard } from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';

import api from "~/services/api";
import getRealm from "~/services/realm";
import Repository from "~/components/Repository";
import {
  Container,
  Title,
  Form,
  Input,
  Submit,
  List } from "./styles";

export default function Main() {
  const [input, setInput] = useState('');
  const [error, setError] = useState(false);
  const [repositories, setRepositories] = useState([]);

  useEffect(() => {
    async function loadRepositories() {
      const realm = await getRealm();

      const data =
      // Ordenar por estrelas
        realm.objects('Repository').sorted("stars", true);

      setRepositories(data);
    }

    loadRepositories();
  }, []);

  async function saveRepository(repository) {
    const data = {
      id: repository.id,
      name: repository.name,
      fullName: repository.full_name,
      description: repository.description,
      stars: repository.stargazers_count,
      forks: repository.forks_count,
    };

    const realm = await getRealm();

    realm.write(() => {
      realm.create('Repository', data, 'modified');
    });

    return data;
  }

  async function handleAddRepository() {
    try {
      const response = await api.get(`/repos/${input}`);

      await saveRepository(response.data);

      setInput('');
      setError(false);
      Keyboard.dismiss();
    } catch (err) {

      setError(true);
    }
  }

  async function handleRefreshRepository(repository) {
    const response = await api.get(`/repos/${repository.fullName}`);

    const data = await saveRepository(response.data);

    setRepositories(repositories.map(repo => repo.id === data.id ? data : repo))
  }

  return (
    <Container>
      <Title>Repositórios</Title>

      <Form>
        <Input
          value={ input }
          error={ error }
          onChangeText={ setInput }
          autoCapitalize="none"
          autoCorrect={false}
          placeholder="Procurar repositório..."
        />

        <Submit onPress={ handleAddRepository }>
          <Icon name="add" size={22} color="#FFF"/>
        </Submit>
      </Form>

      <List
        // Fecha teclado ao selecionar listagem
        keyboardShouldPersistTaps="handled"
        data={ repositories }
        keyExtractor={ item => String(item.id) }
        renderItem={({ item }) => (
          <Repository data={ item }
            onRefresh={ () => handleRefreshRepository(item) }/>
        )}
      />

    </Container>
  );
}
```

<br><br>

## src/components/Repository/index.js 

```js
import React from 'react';
import Icon from 'react-native-vector-icons/FontAwesome';

import {
    Container,
    Name,
    Description,
    Stats,
    Stat,
    StatCount,
    Refresh,
    RefreshText } from "./styles";

export default function Repository({ data, onRefresh }) {
  return (
    <Container>
      <Name>{ data.name }</Name>
      <Description>{ data.description }</Description>

      <Stats>
        <Stat>
          <Icon name="star" size={16} color="#333" />
          <StatCount>{ data.stars }</StatCount>
        </Stat>
        <Stat>
          <Icon name="code-fork" size={16} color="#333" />
          <StatCount>{ data.forks }</StatCount>
        </Stat>
      </Stats>

      <Refresh onPress={ onRefresh }>
        <Icon name="refresh" color="#4169E2" size={16} />
        <RefreshText>ATUALIZAR</RefreshText>
      </Refresh>
    </Container>
  );
}
```

<br><br>

## Interface principal

![1](https://user-images.githubusercontent.com/48495838/69454079-1cc79300-0d44-11ea-9306-eef85cb50e38.JPG)

<br><br>

![2](https://user-images.githubusercontent.com/48495838/69454082-1cc79300-0d44-11ea-941b-a7170af6f198.JPG)
<br>

<br><br>
Renan Borba.

