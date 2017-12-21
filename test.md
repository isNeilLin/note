
## 概览

- `props`调整组件样式
- `attrs`封装组件属性
- `extend`或者`styled(Component)`实现组件样式继承
- 使用`innerRef`代替`ref`
- `ThemeProvider` helper实现主题功能
- `keyframes` helper实现动画界面
- `injectGlobal`设置全局样式

### 基础用法

```javascript
import React, { Component } from 'react';
import styled from 'styled-components';

const Title = styled.h1`
    font-size: 18px;
    text-align: center;
    color: palevioletred;
`
const Wrapper = styled.div`
    padding: 24px;
`
export default class StyledComponents extends Component {
    render(){
        return (
            <Wrapper>
                <Title>This is my first styled-component!</Title>
          </Wrapper>
        )
    }
}
```

### `props`调整组件样式 

```javascript
import React, { Component } from 'react';
import styled from 'styled-components';

const Button = styled.button`
    background: ${props=>props.primary ? 'palevioletred' : 'white'};
    color: ${props=>props.primary ? 'white' : 'palevioletred'};
    font-size: 1em;
	margin: 1em;
	padding: 0.25em 1em;
	border: 2px solid palevioletred;
	border-radius: 3px;
`

export default class StyledComponents extends Component {
    render(){
        return (
            <Wrapper>
                <Button>Normal</Button>
                <Button primary>Primary</Button>
          </Wrapper>
        )
    }
}
```
### `attrs`封装组件属性

```javascript
import React, { Component } from 'react';
import styled from 'styled-components';

const Password = styled.input.attrs({
    // define static props
    type: 'password',
    
    // define dynamic ones
    margin: props => props.size || '1em',
    padding: props => props.size || '1em'
})`
    color: palevioletred;
    font-size: 1em;
    border: 2px solid palevioletred;
    border-radius: 3px;
    margin: ${props=>props.margin};
    padding: ${props=>props.padding};
`
export default class StyledComponents extends Component {
    render(){
        return (
            <Wrapper>
                <Password placeholder="A small text input" size="1em"/>
                <Password placeholder="A bigger text input" size="2em"/>
          </Wrapper>
        )
    }
}
```

### `extend`或者`styled(Component)`实现组件样式继承

```javascript
import React, { Component } from 'react';
import styled from 'styled-components';


const Button = styled.button`
    background: ${props=>props.primary ? 'palevioletred' : 'white'};
    color: ${props=>props.primary ? 'white' : 'palevioletred'};
    font-size: 1em;
	margin: 1em;
	padding: 0.25em 1em;
	border: 2px solid palevioletred;
	border-radius: 3px;
`
// extending styles
// way 1
const TomatoButton = Button.extend`
    color: tomato;
    border-color: tomato;
`
// way 2  better way
const AnotherTomatoButton = styled(Button)`
color: tomato;
border-color: tomato;
`

export default class StyledComponents extends Component {
    render(){
        return (
            <Wrapper>
                <Button>Normal</Button>
                <TomatoButton>TomatoButton</TomatoButton>
                <AnotherTomatoButton>AnotherTomatoButton</AnotherTomatoButton>
          </Wrapper>
        )
    }
}
```

### 使用`innerRef`代替`ref`

```javascript
import React, { Component } from 'react';
import styled from 'styled-components';

const InnerRefInput = styled.input`
    padding: 0.5em;
    margin: 0.5em;
    color: palevioletred;
    background: papayawhip;
    border: none;
    border-radius: 3px;
`

export default class StyledComponents extends Component {
    render(){
        return (
            <Wrapper>
               <InnerRefInput placeholder="hover here"
                innerRef={ x => { this.input = x } }
                onMouseEnter={ () => this.input.focus() }
            />
          </Wrapper>
        )
    }
}
```

### `ThemeProvider` helper实现主题功能

```javascript
import React, { Component } from 'react';
import styled, {ThemeProvider} from 'styled-components';

const ThemeButton = styled.button`
    font-size: 1em;
    margin: 1em;
    padding: 0.25em 1em;
    border-radius: 3px;    
    color: ${props=> props.theme.main};
    border: 2px solid ${props=> props.theme.main};
`
ThemeButton.defaultProps = {
    theme: {
        main: 'palevioletred'
    }
}

const theme = {
    main: 'mediumseagreen'
}


export default class StyledComponents extends Component {
    render(){
        return (
            <Wrapper>
                <ThemeButton>Normal</ThemeButton>
                <ThemeProvider theme={theme}>
                    <div>
                        <ThemeButton>Themed</ThemeButton>
                        <br/>
                    </div>
                </ThemeProvider>
          </Wrapper>
        )
    }
}
```
*非styled-components组件使用theme(主题)*
```javascript
import { withTheme } from 'styled-components'

class MyComponent extends React.Component {
  render() {
    console.log('Current theme: ', this.props.theme);
    // ...
  }
}

export default withTheme(MyComponent)
```

### `keyframes` helper实现动画界面

```javascript
import React, { Component } from 'react';
import styled, {keyframes} from 'styled-components';

const rotate360 = keyframes`
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
`

const Rotate = styled.div`
    padding: 2rem 1rem;
    font-size: 1.2rem;
    display: inline-block;
    animation: ${rotate360} 2s linear infinite;
`
export default class StyledComponents extends Component {
    render(){
        return (
            <Wrapper>
               <Rotate>😊😊😊</Rotate>
          </Wrapper>
        )
    }
}
```

### `injectGlobal`设置全局样式

```javascript
import { injectGlobal } from 'styled-components';

injectGlobal`
  @font-face {
    font-family: 'Operator Mono';
    src: url('../fonts/Operator-Mono.ttf');
  }

  body {
    margin: 0;
  }
`;
```
