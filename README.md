// ScriptCalculator.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"
#include <stack>
#include <list>
#include <iostream>

using namespace std;

enum TokenType
{
    TTInt,
    TTMul,
    TTDiv,
    TTMod,
    TTAdd,
    TTSub,
    TTEnd,
    TTNULL,
};

struct Token
{
    TokenType tokenType;
    long value;
    Token()
    {
        tokenType = TTNULL;
        value = 0;
    }
    Token(TokenType tt, long val)
    {
        tokenType = tt;
        value = val;
    }
    Token(TokenType tt)
    {
        tokenType = tt;
        value = 0;
    }
};

class ScriptCalculator
{
private:
    list<long> operands;
    list<TokenType> operators;
private:
    Token GetToken(char *& script)
    {
        while (*script)
        {
            switch (*script)
            {
            case ' ':
            case '\t':
                script++;
                continue;
            case '+':
                script++;
                return Token(TTAdd);
            case '-':
                script++;
                return Token(TTSub);
            case '*':
                script++;
                return Token(TTMul);
            case '/':
                script++;
                return Token(TTDiv);
            case '%':
                script++;
                return Token(TTMod);
            default:
                {
                    char * pValEnd = script;
                    while (*pValEnd && _istdigit(*pValEnd))
                    {
                        pValEnd++;
                    }
                    char bak = *pValEnd;
                    *pValEnd = 0;
                    long val = atoi(script);
                    *pValEnd = bak;
                    script = pValEnd;
                    return Token(TTInt, val);
                }
            }
        }

        if (*script == NULL) return Token(TTEnd);
        else return Token(TTNULL);
    }
    bool HasHigherPreviousOperator(TokenType currentOp)
    {
        if (currentOp == TTMul || currentOp == TTDiv || currentOp == TTMod) return false;
        return (operators.size() > 0) && (operators.back() == TTMul || operators.back() == TTDiv || operators.back() == TTMod);
    }
    void BinaryOpBack()
    {
        if (operands.size() < 2) throw exception("Invalid script");
        
        long valRight = operands.back(); operands.pop_back();
        long valLeft = operands.back(); operands.pop_back();
        TokenType op = operators.back(); operators.pop_back();

        operands.push_back(BinaryOp(valLeft, op, valRight));
    }
    void BinaryOpFront()
    {
        if (operands.size() < 2) throw exception("Invalid script");

        long valLeft = operands.front(); operands.pop_front();
        long valRight = operands.front(); operands.pop_front();
        TokenType op = operators.front(); operators.pop_front();

        operands.push_front(BinaryOp(valLeft, op, valRight));
    }
    long BinaryOp(long valLeft, TokenType op, long valRight)
    {
        switch (op)
        {
        case TTMul:
            return (valLeft * valRight);
        case TTDiv:
            return(valLeft / valRight);
        case TTMod:
            return(valLeft % valRight);
        case TTAdd:
            return(valLeft + valRight);
        case TTSub:
            return(valLeft - valRight);
        }
    }
public:
    long Eval(const char * script)
    {
        if (script == NULL || strlen(script) < 1) return 0;

        operands.clear();
        operators.clear();

        char * copiedScript = new char[strlen(script) + 1];
        strcpy(copiedScript, script);
        char * pCur = copiedScript;

        Token token;
        while (token.tokenType != TTEnd)
        {
            token = GetToken(pCur);
            switch (token.tokenType)
            {
            case TTInt:
                operands.push_back(token.value);
                break;
            case TTMul:
                operators.push_back(TTMul);
                break;
            case TTDiv:
                operators.push_back(TTDiv);
                break;
            case TTMod:
                operators.push_back(TTMod);
                break;
            case TTAdd:
                if (HasHigherPreviousOperator(token.tokenType)) 
                    BinaryOpBack();
                operators.push_back(TTAdd);
                break;
            case TTSub:
                if (HasHigherPreviousOperator(token.tokenType)) 
                    BinaryOpBack();
                operators.push_back(TTSub);
                break;
            case TTEnd:
                break;
            case TTNULL:
                return LONG_MIN; //INVALID EXPRESSION
            }
        }

        if (HasHigherPreviousOperator(TTAdd))
            BinaryOpBack();

        while (operators.size() > 0)
        {
            BinaryOpFront();
        }

        delete copiedScript;

        return operands.front();
    }
};

int _tmain(int argc, _TCHAR* argv[])
{
    ScriptCalculator sc;
    cout << sc.Eval("1 + 2 + 3 * 4") << endl;
    cout << sc.Eval("12 + 2 * 3 - 4") << endl;

	return 0;
}

