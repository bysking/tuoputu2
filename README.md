// console.log(echarts)
var myChart = echarts.init(document.getElementById('main'));

const list3 = [{
    name: 'a',
    parent: 'b'
}, {
    name: 'a',
    parent: 'c'
}, {
    name: 'b',
    parent: 'd'
}, {
    name: 'c',
    parent: 'd'
},
{
    name: 'd',
    parent: 'g'
},
{
    name: 'b',
    parent: 'e'
},
{
    name: 'c',
    parent: 'f'
},
{
    name: 'e',
    parent: 'g'
},
{
    name: 'f',
    parent: 'g'
}
]
const list = [{
    name: 'a',
    parent: 'c'
}, {
    name: 'a',
    parent: 'b'
}, {
    name: 'b',
    parent: 'd'
}, {
    name: 'c',
    parent: 'd'
},
{
    name: 'd',
    parent: 'e'
},
{
    name: 'e',
    parent: 'f'
},
{
    name: 'e',
    parent: 'g'
},
{
    name: 'f',
    parent: 'h'
},
{
    name: 'f',
    parent: 'i'
},
{
    name: 'g',
    parent: 'j'
},
{
    name: 'g',
    parent: 'k'
},
{
    name: 'i',
    parent: 'l'
},
{
    name: 'j',
    parent: 'l'
},
{
    name: 'l',
    parent: 'm'
},
{
    name: 'h',
    parent: 'm'
},
{
    name: 'k',
    parent: 'm'
},
]

console.log('list')
console.log(list)
// 构造指定key的links数据，echarts要求
const links = list.map((item) => {
    return {
        source: item.name,
        target: item.parent
    }
})

console.log('links')
console.log(links)

// a -> b
// a -> c
// 那么我们得到
// res = {
//     a: ['b', 'c'], // a 的父亲或者孩子
//     b: ['e']
// }
const res = {}
list.forEach((item) => {
    const { name, parent } = item
    if (!res[name]) {
        res[name] = []
        res[name].push(parent)
    } else {
        res[name].push(parent)
    }
})

console.log('构建父子数组')
console.log(res)
// root表示给定的根节点以及数据结构，level表示横向层级，y表示纵向层级,两个用来计算坐标
const root = {
    name: 'a',
    parent: [],
    level: 1,
    y: 1
}

// res = {
//     a: ['b', 'c'], // a 的父亲或者孩子
//     b: ['e']
// }
// root默认是a那么他的父亲从上面计算的res里面拿
root.parent = res[root.name]

// console.log(root)
res2 = [] // 计算结果统计
res2.push(root)
fn(root, root.parent)
console.log('计算层级')
console.log(res2)

// 定义计算指定节点的所有父亲节点的横向层级level函数
function fn(root, parent) { // root根节点，parent: 根节点父亲
    if (!parent || parent.length == 0) return // 没有父亲就终止
    parent.forEach((item, index) => {
        let single = true
        res2.forEach((ite) => { // 保证res2中不重复
            if (ite.name == item) {
                single = false
            }
        })
        single && res2.push({ // 计算level结果统计
            name: item,
            parent: res[item],
            level: root.level + 1 // 计算的level
        })
    })
    parent.forEach((item) => {
        let itemobj = res2.findIndex((i) => {
            return i.name == item
        })
        itemobj = res2[itemobj]
        fn(itemobj, itemobj.parent)
    })
}

// 计算高度层级
const map = {} // 存放level对应的节点个数，每个level对应的节点数用来依次给该层y赋值
const temp = res2.map((ite) => {
    if (!map[ite.level]) { // 第一次统计到level
        map[ite.level] = 1 // set 1
    } else {
        map[ite.level] = map[ite.level] + 1 // 统计到就加一
    }
    ite.y = map[ite.level] // 每个level对应的节点数用来依次给该层y赋值
    return ite // 返回ite, 上面操作的都是对象引用，注意bug
})
// console.log('修正高度层级')
// console.log(temp)

const width = 600 // 宽
const height = 400 // 高
const n = 4 //level的最大值
const j = 2 // y的最大值
const xx = width / n // 横向一单位长度
const yy = height / (j * 2) // 纵向一单位长度

const res3 = temp.map((item) => { // 构造data数组，和links一起传给echarts
    return {
        name: item.name,
        x: item.level * xx, // 横坐标
        y: yy * (2 * item.y - 1) // 纵坐标，*2是因为两节点平分中间间距
    }
})
console.log('计算坐标')
console.log(res3)

option = {
    title: {
        text: 'Graph 简单示例'
    },
    tooltip: {},
    animationDurationUpdate: 1500,
    animationEasingUpdate: 'quinticInOut',
    series: [
        {
            type: 'graph',
            layout: 'none',
            symbolSize: 50,
            roam: true,
            label: {
                normal: {
                    show: true
                }
            },
            edgeSymbol: ['circle', 'arrow'],
            edgeSymbolSize: [4, 10],
            edgeLabel: {
                normal: {
                    textStyle: {
                        fontSize: 20
                    }
                }
            },
            data: res3,
            links: links,
            lineStyle: {
                normal: {
                    opacity: 0.9,
                    width: 2,
                    curveness: 0
                }
            }
        }
    ]
};

myChart.setOption(option)
